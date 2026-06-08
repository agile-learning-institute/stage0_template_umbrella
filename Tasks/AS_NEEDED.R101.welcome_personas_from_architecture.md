# R101 – Welcome page personas and JWT roles from architecture

**Status**: As Needed  
**Task Type**: Refactor  
**Run Mode**: Run as needed

## Goal

Align developer sign-in **persona** data with `Specifications/architecture.yaml`. JWT minting lives in **`login.html`** and **`welcome-auth.js`**; `index.html` is catalog-only (no persona matrix).

Default personas: **Carol** (`coordinator`), **Maria** (`mentor`), **Cat** (`customer`), **Mark** (`mentee`), **Stan** (`admin` / SRE)—five personas, five roles. Run this task when architecture or persona definitions change.

## Context / Input files

**Inputs** (read first):

- `Specifications/architecture.yaml` — domains where `is_journey: true`, repo `type: spa` (ports), and any fields you add for personas (see below).
- `login.html` and `welcome-auth.js` — persona dropdown, role checkboxes, `return_to` allowlist, HS256 minting.
- API **e2e / dev defaults**: `JWT_SECRET`, `JWT_ISSUER`, `JWT_AUDIENCE`, `JWT_ALGORITHM` must match token claims (see flask mongo template `test/e2e/e2e_auth.py` and Pipfile `dev` / `e2e` scripts).

**Optional** (only if you extend the spec):

- Product slug / naming: `Specifications/product.yaml` — affects repo naming, not JWT directly.

## Requirements

1. **Drive SPA `return_to` allowlist from architecture**  
   Keep journey SPA origins in `welcome-auth.js` aligned with `architecture.yaml` ports so Login redirects only to known local SPAs.

2. **Replace or customize default personas**  
   Carol, Maria, Cat, Mark, Stan ship in the template; change labels, `sub`, or default roles when your product differs.

3. **JWT and URL `roles` must agree**  
   `login.html` mints JWTs at Login from the user dropdown + role checkboxes. Payload `roles` array must match hash `roles` (comma-separated). Users may override checkboxes before Login.

4. **Mint tokens with the real dev secret**  
   Use the same secret as running APIs (e.g. PyJWT, `HS256`). Default Developer Edition and compose use `local-dev-jwt-secret-fixed` (no product slug in the string). After changing `JWT_SECRET` or claims, re-run this task and update persona data in `welcome-auth.js`.

5. **Document in-repo**  
   Add a short comment in `welcome-auth.js` describing iss/aud/sub/roles and that tokens are **dev-only**.

6. **Do not break non-persona sections**  
   `index.html` service list, API Explorer, and `spa_ref` URL wiring should stay correct for `schema` and journey domains.

## Suggested approaches

- **Minimal static map in `welcome-auth.js`**: Persona list keyed by journey hints; mint client-side when users click Login.  
- **Spec-driven (future)**: Add explicit optional fields under each journey domain for persona labels, then generate `welcome-auth.js` from Jinja — only if your team wants YAML to own that data.

## Testing expectations

- Open `login.html` locally (`welcome` service / port from Developer Edition).  
- For each journey SPA, sign in as the matching persona (Cat → customer, Carol → coordinator, Maria → mentor, Mark → mentee, Stan → admin routes).  
- APIs accept tokens when `JWT_SECRET` matches; use Stan for admin-gated endpoints.  
- After changes, run umbrella `make test` if you are working in the template repo.

## Dependencies / Ordering

- Run after merge when `architecture.yaml` lists final journey domains and ports.  
- Complements **R100** (compose + welcome layout/services); R100 updates structure for new services; **R101** focuses on **persona UX and JWTs**.

## Change control checklist

- [ ] Read current `architecture.yaml` journey domains and SPA ports.  
- [ ] Confirm personas and roles match your product.  
- [ ] Update `welcome-auth.js` (and `login.html` if UI changes).  
- [ ] Manually verify one SPA per journey.  
- [ ] Note completion and date in implementation notes below.

## Implementation notes

_(Fill in when task is executed.)_

## Testing results

_(Fill in when task is executed.)_
