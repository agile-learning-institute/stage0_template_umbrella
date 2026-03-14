# Stage0 automation

This folder contains Makefile targets for **launch**, **clone**, and **delete** workflows that create and manage service repos from templates. Run **from this folder**; specifications are read from `../../Specifications` (architecture.yaml, product.yaml).

## Prerequisites

- **GITHUB_TOKEN** – Classic token with `repo`, `workflow`, `write:packages` (and `delete:packages`, `delete_repo` for delete commands). Store in `~/.mentorhub/GITHUB_TOKEN`.
- **gh** – [GitHub CLI](https://cli.github.com/) installed and authenticated.
- **yq** – [yq](https://mikefarah.gitbook.io/yq) for reading `architecture.yaml` and `product.yaml`.
- **Docker** – Logged in to `ghcr.io` (e.g. via `make update` from repo root). **Docker Buildx** is used for multi-arch builds in CI; local single-arch builds do not require it.
- **Git** – Configured for push (HTTPS with token or SSH). Global `user.name` and `user.email` recommended for commits.

For CLI install, token creation, and Docker login, see [CONTRIBUTING.md](../../CONTRIBUTING.md).

## Usage

Run from **this folder** (DeveloperEdition/stage0):

```sh
cd DeveloperEdition/stage0   # from repo root

# Validate all prerequisites (build tools, gh, ssh, buildx, git config, Git SSH)
make validate

# Launch all services (create repos from templates, merge, build, publish, push)
make launch-all

# Launch specific domains only
make launch-services SERVICES="schema sample"

# Clean, clone, and build (no publish; useful after manual changes)
make clean-clone-build SERVICES="sample"

# Delete services (repos and packages; DESTRUCTIVE)
make delete-services SERVICES="sample"
make delete-all
```

## What the targets do

- **launch-all** – Builds and pushes the umbrella container, then for each domain in `Specifications/architecture.yaml`: creates GitHub repos from templates, clones them, runs merge with your specs, builds/publishes where configured, and pushes to `main`. The push to `main` triggers CI to build multi-arch images.
- **launch-services** – Same as above but only for the listed `SERVICES` (space-separated domain names).
- **clean-clone-build** – Removes local clones, re-clones from GitHub, and runs build (no publish). Use when you want a fresh clone/build without creating new repos.
- **delete-services** / **delete-all** – Deletes GitHub repos and (where applicable) packages for the given domains. **Not reversible.**

## Architecture

Targets read `../../Specifications/architecture.yaml` and `../../Specifications/product.yaml`. Service repos are created under the **parent** of the umbrella repo (e.g. if the umbrella is `~/repos/myproduct`, repos are in `~/repos/` as `myproduct_runbook_api`, etc.).