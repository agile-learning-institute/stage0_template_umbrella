# SRE Standards

## Tech Stack
- Source Control: Github 
- CI Automation: Github Actions
- Private Container Registry: GitHub Container Registry till AWS refactor
- Private PyPi Registry: GitHub via https+token till JFrog refactor
- Private NPM Registry: GitHub Packages till JFrog refactor (SPA shared libraries also install cleanly from **Git** without publishing a package artifact; see [SPA Standards](./spa_standards.md))
- Infrastructure Automation: Docker Compose till Terraform refactor
- Container Hosting: AWS EKS?
- Container Configuration: Helm? 
- Container Orchestration: Argo CD?
- Monitoring: Prometheus, Grafana, ELK
- Runbook Automation: [stage0 runbooks](https://github.com/agile-learning-institute/stage0_runbooks)

## Developer Experience
The ``mh`` Developer Edition CLI is how SRE provides a strong developer experience. It manages developer environment values (keys, secrets, JWT material for local tooling, etc.) and wraps the services configured in this [docker-compose](../docker-compose.yaml) file. Developers can run the full stack on local hardware without using `docker compose` directly for normal workflows.

**Local authentication (Developer Edition):** APIs do **not** register per-service `/dev-login`. Developer Edition compose uses a **stable `JWT_SECRET`** and keeps **`ENABLE_LOGIN` off** for APIs so SPAs and backends agree across restarts. The umbrella **welcome page** (`index.html`) drives dev sign-in: it navigates to each SPA with URL-hash bootstrap parameters (`access_token`, `expires_at`, `roles`). SPAs call **`bootstrapDevAuthFromUrl`** from shared SPA utilities before boot so `localStorage` matches production-style bearer usage. **`IDP_LOGIN_URI`** / **`VITE_IDP_LOGIN_URI`** default to the welcome page origin (for example `http://127.0.0.1:8080/`) so unauthenticated guards, `401` handling, and logout send users back to that page—not to a per-SPA `/login` route.

**Verifying the stack after compose or image changes** (from the product checkout root, for example the repo that contains `DeveloperEdition/`):

```sh
cd mentorhub
make update
mh up all
```

## Production alignment

**API gateway and commercial IdP:** In production, traffic is intended to sit behind an **API gateway** (or edge proxy) with **TLS**, routing to SPA static assets and API services. **Authentication** uses a **commercial IdP** (OAuth2/OIDC). Access tokens are issued by the IdP (or a BFF); applications do not mint tokens via `/dev-login`. APIs validate JWTs (shared secret or JWKS) with the same claim expectations as in developer edition. SPAs redirect to the real IdP login/authorize entry via the configured login base URL—preserving a single auth story from local welcome page through to production IdP.

## SRE Automation 
SRE Automation is done using the [stage0 runbooks](https://github.com/agile-learning-institute/stage0_runbooks) system. Our custom runbook is [runbook_api](https://github.com/agile-learning-institute/mentorhub_runbook_api) which is available for use with ``mh up runbook`` and accessing http://localhost and following the runbooks link. 

## Continuous Integration
The developer workflow follows the feature branch pattern. A developer creates a branch to work on a feature, and submit a pull request (PR) when the feature is ready to be deployed. When a PR is approved by a reviewer and merged to the main branch, the CI automation will build and push a new container with a :latest tag to the system's container registry. These containers are deployed to a cloud DEV environment, and available for developers to use for local development.

NOTE: We are using ``ghcr`` as our container registry at this time. We will shift to an AWS container registry when we are ready to start cloud based deployments. 

## Continuous Deployment
Infrastructure provisioning is automated using ?Terraform?. Deployment of code through different environments is managed using container tagging. TBD Run book automation implements continuous deployment actions such as "Provision a Training Environment", "Run Regression Testing in the TEST environment", "Promote all containers from TEST to STAGING" or "Restore Production Database backups to Staging Database"

## API Reverse Proxy
All SPA's are served by NGINX with reverse proxy configuration for API endpoints. This allows for secure networking configurations that do not expose the API to external access, establishing a clear separation between the front end and back end networks.

### NGINX Configuration Pattern
SPA containers use an NGINX configuration template (`nginx.conf.template`) that is processed at container startup using `envsubst`. The template supports the following environment variables:

- **`API_HOST`**: Hostname of the API server (default: `localhost`)
- **`API_PORT`**: Port of the API server (default: `8083`)
- **`IDP_LOGIN_URI`**: Full base URL for login redirect after logout, on `401`, or when the SPA is not authenticated (Developer Edition default: umbrella welcome page, e.g. `http://127.0.0.1:8080/`; production: IdP or gateway login entry)

Build-time SPA env (**`VITE_IDP_LOGIN_URI`**) should match the same logical URL so the client can redirect without relying on NGINX-only rewrites.

### Reverse Proxy Routes
The NGINX configuration proxies the following routes to the API server:

- **`/api/*`**: All API endpoints are proxied to `http://${API_HOST}:${API_PORT}/api/`

Do not proxy `/dev-login`; that endpoint is not part of the product API surface.

### Authentication Redirect Pattern
Protected routes and the API client redirect the browser to the configured **login base URL** (`getIdpLoginBaseUrl()` / `VITE_IDP_LOGIN_URI`) when the user is unauthenticated or tokens are cleared:

- **Developer Edition:** Points at the umbrella welcome page so developers pick a persona and land in the SPA with hash bootstrap.
- **Production:** Points at the commercial IdP (or gateway-hosted login) with TLS.

This keeps one redirect contract from local through production without per-SPA `/login` pages. 

## Service Configurability
All API's are configured using a shared [Config singleton](https://github.com/agile-learning-institute/api_utils/blob/main/py_utils/config/config.py). The Config object manages all configuration items for all API and SPA code. Configuration values are read from the first of: Config File, Environment Var, Default Value. The configuration items and non-secret values are exposed through the Config API endpoint, which is used by the SPA to get runtime configuration values.

## Service Observability
All API's expose a /metrics endpoint which exposes a text-based exposition format that Prometheus understands. This endpoint exposes detailed, real-time metrics about the API's performance, latency, error rates, and internal health.

## API Security Standards

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

| Feature | Developer Edition | Production |
|---------|-------------------|------------|
| `ENABLE_LOGIN` on APIs | `false` (no dev-login blueprint) | `false` (never enable) |
| `JWT_SECRET` | Stable value in compose (aligns CLI/local JWT tooling) | Strong random / secrets manager |
| Token issuance | Welcome page / dev tokens; hash bootstrap into SPAs | Commercial IdP (or BFF) |
| Token validation | Full signature verification | Full signature verification |
| `/dev-login` | Not exposed | Not present |
| SPAs | No `/login` route; redirect to login base URL | Same; IdP URL |
| Logging | INFO or DEBUG | WARNING or ERROR |

## API Container Configuration
- Dockerfile must define `API_HOST` and `API_PORT` environment variables
- NGINX configuration template (`nginx.conf.template` or `default.conf.template`) must use `${API_HOST}` and `${API_PORT}` in proxy_pass directive
- Template pattern: `proxy_pass http://${API_HOST}:${API_PORT}/api/;`
- NGINX automatically substitutes environment variables from templates in `/etc/nginx/templates/`
- Container exposes port 80 by default (or `SPA_PORT` if specified)