# API Standards

## Technology Stack
- Python v3.15
- pipenv v2026.0.2
- Flask (current) 
- pymongo v4.15.5
- PyJWT for JWT token handling with signature verification
- prometheus-flask-exporter for metrics
- pytest for unit testing
- pytest-cov for code coverage
- requests for E2E testing

## Dependency Management
- All dependencies are managed via `Pipfile` and `Pipfile.lock`
- The `api_utils` shared library is installed via HTTPS from GitHub using Personal Access Tokens (PATs)
- Docker builds use `GITHUB_TOKEN` build argument for authentication
- Local development requires git credential configuration (see Developer Edition README)

## Standard Developer Commands
- pipenv run build (package code for deployment)
- pipenv run dev (run dev server)
- pipenv run db (start backing db container)
- pipenv run api (start db + api containers)
- pipenv run service (start db, api, spa containers)
- pipenv run container (build API container)

## API Design
- Create, Retrieve, Patch design pattern
- API's work with a model-less document management approach
- Open API Specification (swagger) is a Design Specification, NOT a code build artifact
- Route blueprints use factory functions (e.g., `create_*_routes()`) that return Flask Blueprints
- Route registration should be grouped together in `server.py` for clarity

## Server.py Organization Pattern
All API servers should follow the organizational pattern established in api_utils/server.py:

1. **Module docstring** - Describe the server purpose and capabilities
2. **Standard library imports** - `sys`, `os`, `signal`, Flask imports
3. **Config singleton initialization** - Initialize before logging
4. **Logging setup** - After Config initialization
5. **MongoIO singleton and configuration** - Set enumerators and versions
6. **Flask app initialization** - Create app with MongoJSONEncoder
7. **Route imports (grouped together)** - All route factory imports in one place
8. **Route registration (grouped together)** - All blueprint registrations together
9. **Logging summary** - Clear summary of registered routes
10. **Signal handlers** - SIGTERM and SIGINT for graceful shutdown
11. **Main entry point** - `if __name__ == "__main__"` block

## API Documentation (Explorer)
All APIs must provide interactive API documentation using Swagger UI:

- **Location**: `docs/explorer.html` in the repository root
- **Source**: Copy from api_utils/docs/explorer.html
- **Customization**: Update the `<title>` tag to reflect your API name (e.g., "Template API Explorer")
- **OpenAPI Spec**: Must provide `docs/openapi.yaml` file
- **Access**: Serve at `/docs` endpoint

## Separation of Concerns
- `server.py` is the standard API entry point
- `command.py` is the standard CLI entry point
- `/routes/*domain*_routes.py` handle HTTP request/response logic
- `/services/*domain*_service.py` handles business logic/RBAC for domain
- See api_utils for shared utilities

### Required Utilities from api_utils
All APIs must use the following utilities from the `api_utils` package:

- **Config Singleton**: Use `Config.get_instance()` for all configuration values
  - Configuration follows precedence: Config File → Environment Variable → Default Value
  - Non-secret values are exposed via `/api/config` endpoint
  
- **MongoIO Singleton**: Use `MongoIO.get_instance()` for all MongoDB operations
  - Provides connection pooling and error handling
  - Configures enumerators and versions on initialization

- **Flask Utilities**:
  - `create_flask_token()` - Extract and validate JWT tokens from Authorization header
  - `create_flask_breadcrumb(token)` - Generate request breadcrumbs for logging
  - `handle_route_exceptions` - Decorator for consistent exception handling
  - `MongoJSONEncoder` - Custom JSON encoder for MongoDB document types
  - Custom exceptions: `HTTPUnauthorized`, `HTTPForbidden`, `HTTPNotFound`, `HTTPInternalServerError`
  - **Security**: Do not include PII or User Data in exceptions

### Required Endpoints
All APIs must implement the following standard endpoints:

- **`/metrics`** - Prometheus metrics endpoint (use `create_metric_routes(app)`)
- **`/api/config`** - Configuration endpoint (use `create_config_routes()`)
- **`/dev-login`** - Development JWT token issuance (use `create_dev_login_routes()`)
  - Returns 404 when `ENABLE_LOGIN=False` (default - hides endpoint existence)
  - **NEVER** enable in production environments
  - Endpoint is NOT reverse-proxied for security (networking prevents direct access)
  - See SECURITY.md for production authentication setup
- **`/docs/*`** - API explorer/OpenAPI documentation (use `create_explorer_routes()`)

### Authentication Pattern
APIs implement JWT-based authentication with the following patterns:

- **JWT Secret Requirement (CRITICAL)**: `JWT_SECRET` must be explicitly set before starting any API. Applications will fail fast on startup if `JWT_SECRET` uses the default value. This prevents accidental production deployments with insecure secrets.
  ```bash
  # Development (de script sets this automatically)
  export JWT_SECRET='timestamp-based-dev-secret'
  
  # Production (use strong random value)
  export JWT_SECRET=$(openssl rand -base64 32)
  ```

- **JWT Secret Management**: The `de` script automatically generates a timestamp-based `JWT_SECRET` on each execution. This ensures tokens are invalidated after server restarts, requiring re-authentication after deployments.
  
- **Token Validation**: Use `create_flask_token()` in protected routes to automatically validate JWT tokens from the `Authorization: Bearer <token>` header. Tokens are validated with:
  - Signature verification using `JWT_SECRET`
  - Expiration check
  - Issuer and audience validation
  
- **Dev-Login Endpoint**: The `/dev-login` endpoint issues signed JWTs for local/development environments. 
  - Controlled by `ENABLE_LOGIN` configuration (default: false)
  - Returns 404 when disabled (hides endpoint existence for security)
  - **NEVER** enable in production environments
  - Endpoint is NOT reverse-proxied (networking prevents direct access in production)

- **Protected Routes**: Use `@handle_route_exceptions` decorator and `create_flask_token()` to protect routes:
  ```python
  @route.route('/protected', methods=['GET'])
  @handle_route_exceptions
  def protected_route():
      token = create_flask_token()  # Validates JWT signature, raises HTTPUnauthorized if invalid
      breadcrumb = create_flask_breadcrumb(token)
      # ... route logic
  ```

### Standard Route Registration Pattern
All route blueprints should be registered in `server.py` following this pattern:

```python
from api_utils import (
    create_metric_routes,
    create_explorer_routes,
    create_dev_login_routes,
    create_config_routes
)

# Register route blueprints
app.register_blueprint(create_explorer_routes(), url_prefix='/docs')
app.register_blueprint(create_dev_login_routes(), url_prefix='/dev-login')
app.register_blueprint(create_config_routes(), url_prefix='/api/config')
metrics = create_metric_routes(app)  # Middleware, not a blueprint

# Register domain-specific routes
app.register_blueprint(create_grade_routes(), url_prefix='/api/grade')
app.register_blueprint(create_testrun_routes(), url_prefix='/api/testrun')
```

**Note**: `create_metric_routes()` is middleware that wraps the Flask app directly, not a blueprint. Do not use `app.register_blueprint()` with it.

## Security Standards

### Production Requirements

Before deploying any API to production, ensure:

- [ ] `JWT_SECRET` is set to a strong, randomly generated value (not default)
- [ ] `ENABLE_LOGIN` is set to `false` or not set (default is false)
- [ ] MongoDB connection uses authentication and encryption
- [ ] HTTPS/TLS is configured via reverse proxy
- [ ] Monitoring and logging are enabled
- [ ] All dependencies are up to date

### JWT Security

- **Signature Verification**: api_utils validates JWT signatures when `JWT_SECRET` is configured
- **Fail-Fast Validation**: Applications will not start with default `JWT_SECRET` value
- **Token Requirements**: All tokens must include `iss`, `aud`, `sub`, `exp` claims
- **Secret Rotation**: Plan for regular secret rotation in production environments

### Development vs Production

| Feature | Development | Production |
|---------|-------------|------------|
| `ENABLE_LOGIN` | `true` (set by de script) | `false` (never enable) |
| `JWT_SECRET` | Timestamp-based | Strong random value |
| Token Validation | Full signature verification | Full signature verification |
| `/dev-login` | Available at localhost | 404 (disabled) |
| Logging | INFO or DEBUG | WARNING or ERROR |

