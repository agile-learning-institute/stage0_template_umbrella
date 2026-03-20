# --- Configuration (SLUG/ORG from product.yaml) ---
MAKEFILE_DIR := .
PRODUCT_FILE ?= $(MAKEFILE_DIR)/Specifications/product.yaml
SLUG := $(shell yq -r '.info.slug' $(PRODUCT_FILE))
ORG := $(shell yq -r '.organization.git_org' $(PRODUCT_FILE))
LOG_FILE ?= .make.log
SPECS_DIR := $(CURDIR)/Specifications
MERGE_IMAGE := ghcr.io/agile-learning-institute/stage0_runbook_merge:latest

.PHONY: help install update verify schemas container push build-package publish-package \
	launch-all launch-services clone-all delete-all delete-services stage0-automation-help

help:
	@echo "Mentor Hub Developer CLI - Available commands:"
	@echo ""
	@echo "  make install        - Install mentorhub CLI tools to ~/.mentorhub"
	@echo "  make verify        - Verify build tools and prerequisites"
	@echo "  make update        - Update mentorhub CLI tools and configure Docker/Git"
	@echo "  make schemas       - Fetch JSON schemas for all data dictionaries, assumes mongodb_api is running"
	@echo "  make build-package - Build the Mentor Hub welcome page Docker container locally"
	@echo "  make stage0-automation-help - Service repo launch/clone/delete via stage0_launch container"
	@echo ""
	@echo "For more information, see ./CONTRIBUTING.md"

verify:
	@fail=0; \
	echo "=== Verifying installed tools ==="; \
	echo ""; \
	echo "--- Build tools ---"; \
	command -v make >/dev/null 2>&1 && printf "make:    " && make --version | head -1 || { echo "  FAIL: make"; fail=1; }; \
	command -v node >/dev/null 2>&1 && printf "node:    " && node --version || { echo "  FAIL: node"; fail=1; }; \
	command -v npm >/dev/null 2>&1 && printf "npm:     " && npm --version || { echo "  FAIL: npm"; fail=1; }; \
    (vite --version 2>/dev/null || npx vite --version 2>/dev/null) >/dev/null && printf "vite:    " && (vite --version 2>/dev/null || npx vite --version 2>/dev/null) || { echo "  FAIL: vite"; fail=1; }; \
	echo ""; \
	echo "--- Python tools (3.12 required for projects) ---"; \
	PY312=$$(command -v python3.12 2>/dev/null); \
	if [ -n "$$PY312" ] && $$PY312 -c "import sys; exit(0 if sys.version_info[:2] == (3, 12) else 1)" 2>/dev/null; then \
		printf "python3.12: " && $$PY312 --version; \
	else \
		echo "  python3.12: not in PATH (pipenv may use pyenv)"; \
	fi; \
	command -v pipenv >/dev/null 2>&1 && printf "pipenv:  " && pipenv --version || { echo "  FAIL: pipenv"; fail=1; }; \
	PYTEST=$$(mktemp -d); \
	if (cd "$$PYTEST" && pipenv --python 3.12 install >/dev/null 2>&1 && pipenv run python -c "import sys; exit(0 if sys.version_info[:2] == (3, 12) else 1)" >/dev/null 2>&1); then \
		echo "  pipenv+3.12: OK (projects will use 3.12 venv)"; \
	else \
		echo "  FAIL: pipenv cannot use Python 3.12 (install pyenv + 3.12 or python3.12)"; fail=1; \
	fi; \
	rm -rf "$$PYTEST"; \
	echo ""; \
	echo "--- Container tools ---"; \
	command -v docker >/dev/null 2>&1 && printf "docker:  " && docker --version || { echo "  FAIL: docker"; fail=1; }; \
	echo ""; \
	echo "--- GitHub & Git ---"; \
	[ -n "$${GITHUB_TOKEN:-}" ] && printf "GITHUB_TOKEN: set\n" || { echo "  FAIL: GITHUB_TOKEN (set env var)"; fail=1; }; \
	command -v git >/dev/null 2>&1 && printf "git:     " && git --version || { echo "  FAIL: git"; fail=1; }; \
	echo "Checking git global user.name and user.email (recommended)..."; \
	git config --global user.name >/dev/null 2>&1 && echo "  user.name: set" || echo "  user.name: not set (recommended for commits)"; \
	git config --global user.email >/dev/null 2>&1 && echo "  user.email: set" || echo "  user.email: not set (recommended for commits)"; \
	echo ""; \
	echo "--- Utilities ---"; \
	command -v jq >/dev/null 2>&1 && printf "jq:      " && jq --version || { echo "  FAIL: jq"; fail=1; }; \
	command -v yq >/dev/null 2>&1 && printf "yq:      " && yq --version || { echo "  FAIL: yq"; fail=1; }; \
	command -v curl >/dev/null 2>&1 && printf "curl:    " && curl --version | head -1 || { echo "  FAIL: curl"; fail=1; }; \
	echo ""; \
	if [ $$fail -eq 1 ]; then \
		echo "Some prerequisites are missing. See CONTRIBUTING.md for install instructions."; \
		exit 1; \
	fi; \
	echo "=== All prerequisites verified ==="

install:
	@echo "Installing mentorhub CLI..."
	@mkdir -p ~/.mentorhub
	@if ! grep -q "Added by mentorhub CLI install" ~/.zshrc 2>/dev/null; then \
		echo "\n# Added by mentorhub CLI install" >> ~/.zshrc; \
		echo "export PATH=\$$PATH:~/.mentorhub" >> ~/.zshrc; \
		echo "export GITHUB_TOKEN=\$$(cat ~/.mentorhub/GITHUB_TOKEN)" >> ~/.zshrc; \
		echo "Added ~/.mentorhub to PATH in ~/.zshrc"; \
	else \
		echo "~/.mentorhub already in PATH"; \
	fi
	@echo "Installation complete. Run 'source ~/.zshrc' or restart your terminal."

uninstall:
	@echo "Uninstalling mentorhub CLI..."
	@if [ -f ~/.zshrc ]; then \
		grep -v -e 'Added by mentorhub CLI install' \
			-e 'export PATH=.*~/.mentorhub' \
			-e 'export GITHUB_TOKEN=.*mentorhub/GITHUB_TOKEN' \
			~/.zshrc > ~/.zshrc.tmp && mv ~/.zshrc.tmp ~/.zshrc && \
		echo "Removed mentorhub lines from ~/.zshrc"; \
	else \
		echo "~/.zshrc not found, skipping"; \
	fi
	@rm -rf ~/.mentorhub && echo "Removed ~/.mentorhub"
	@echo "Uninstall complete. Run 'source ~/.zshrc' or restart your terminal."

update: verify
	@echo "Updating mentorhub CLI..."
	@if [ ! -f ~/.mentorhub/GITHUB_TOKEN ]; then \
		echo "Error: GITHUB_TOKEN not found! - See ./DeveloperEdition/README.md"; \
		exit 1; \
	fi
	@cp ./DeveloperEdition/mh ~/.mentorhub/mh && \
	chmod +x ~/.mentorhub/mh && \
	cp ./DeveloperEdition/docker-compose.yaml ~/.mentorhub/docker-compose.yaml && \
	GITHUB_TOKEN=$$(cat ~/.mentorhub/GITHUB_TOKEN) && \
	echo "$$GITHUB_TOKEN" | docker login ghcr.io -u $(ORG) --password-stdin && \
	echo "Docker login completed" && \
	git config --global --unset-all url."https://@github.com/".insteadOf 2>/dev/null || true && \
	git config --global url."https://x-access-token:$$GITHUB_TOKEN@github.com/".insteadOf "https://github.com/" && \
	echo "Git URL configured" && \
	echo "Updates completed"

schemas:
	@echo "Fetching JSON schemas for all data dictionaries..."
	@mkdir -p ./Specifications/schemas
	@yq -r '.data_dictionaries[].name' ./Specifications/catalog.yaml | \
	while IFS= read -r name; do \
		[ -z "$$name" ] && continue; \
		echo "Fetching schema for $${name}"; \
		curl -s "localhost:8180/api/configurations/json_schema/$${name}.yaml/0.1.0.0" > "./Specifications/schemas/$${name}.schema.json" \
		|| echo "Warning: Failed to fetch schema for $${name}"; \
	done
	@echo "Schema fetching complete."

container:
	@echo "Building Mentor Hub container..."
	@DOCKER_BUILDKIT=0 docker build -t ghcr.io/agile-learning-institute/mentorhub:latest .
	@echo "Container built successfully: ghcr.io/agile-learning-institute/mentorhub:latest"

push:
	@echo "Pushing Mentor Hub container..."
	@docker push ghcr.io/agile-learning-institute/mentorhub:latest
	@echo "Container Pushed successfully: ghcr.io/agile-learning-institute/mentorhub:latest"

build-publish: container push

build-package: container
publish-package: push
delete-package:
	@gh api -X DELETE /orgs/agile-learning-institute/packages/container/mentorhub

# --- Stage0 automation (image: ghcr.io/.../stage0_launch:latest; needs Docker + GITHUB_TOKEN) ---
STAGE0_IMAGE := ghcr.io/agile-learning-institute/stage0_launch:latest
STAGE0_UMBRELLA_HOST_PATH ?= $(CURDIR)
STAGE0_LAUNCHPAD_HOST_PATH ?= $(shell dirname "$(CURDIR)")

stage0-automation-help:
	@echo "Stage0 automation via $(STAGE0_IMAGE):"
	@echo "  make launch-all              - All service repos (create/merge/build/publish)"
	@echo "  make launch-services SERVICES=\"domain ...\""
	@echo "  make clone-all               - Clone all service repos beside umbrella + build (no publish)"
	@echo "  make delete-services SERVICES=\"...\"  - Confirm in TTY, or set I_CONFIRM_DELETE_SERVICES=yes I_CONFIRM_SLUG=<slug>"
	@echo "  make delete-all              - All services + umbrella repo/package; I_CONFIRM_DELETE_ALL=yes I_CONFIRM_SLUG=<slug>"
	@echo "  Export GITHUB_TOKEN. Override paths: STAGE0_UMBRELLA_HOST_PATH STAGE0_LAUNCHPAD_HOST_PATH"

launch-all:
	@[ -n "$$GITHUB_TOKEN" ] || (echo "Error: export GITHUB_TOKEN"; exit 1)
	@STAGE0_UMBRELLA_HOST_PATH="$(STAGE0_UMBRELLA_HOST_PATH)" STAGE0_LAUNCHPAD_HOST_PATH="$(STAGE0_LAUNCHPAD_HOST_PATH)" \
		docker compose -f DeveloperEdition/launch-docker-compose.yaml pull automation 2>/dev/null || true
	@STAGE0_UMBRELLA_HOST_PATH="$(STAGE0_UMBRELLA_HOST_PATH)" STAGE0_LAUNCHPAD_HOST_PATH="$(STAGE0_LAUNCHPAD_HOST_PATH)" \
		docker compose -f DeveloperEdition/launch-docker-compose.yaml run --rm \
		automation launch-all

launch-services:
	@[ -n "$$GITHUB_TOKEN" ] || (echo "Error: export GITHUB_TOKEN"; exit 1)
	@[ -n "$(SERVICES)" ] || (echo "Error: SERVICES required (e.g. SERVICES=\"schema sample\")"; exit 1)
	@STAGE0_UMBRELLA_HOST_PATH="$(STAGE0_UMBRELLA_HOST_PATH)" STAGE0_LAUNCHPAD_HOST_PATH="$(STAGE0_LAUNCHPAD_HOST_PATH)" \
		docker compose -f DeveloperEdition/launch-docker-compose.yaml pull automation 2>/dev/null || true
	@STAGE0_UMBRELLA_HOST_PATH="$(STAGE0_UMBRELLA_HOST_PATH)" STAGE0_LAUNCHPAD_HOST_PATH="$(STAGE0_LAUNCHPAD_HOST_PATH)" \
		docker compose -f DeveloperEdition/launch-docker-compose.yaml run --rm \
		-e SERVICES="$(SERVICES)" \
		automation launch-services

clone-all:
	@[ -n "$$GITHUB_TOKEN" ] || (echo "Error: export GITHUB_TOKEN"; exit 1)
	@STAGE0_UMBRELLA_HOST_PATH="$(STAGE0_UMBRELLA_HOST_PATH)" STAGE0_LAUNCHPAD_HOST_PATH="$(STAGE0_LAUNCHPAD_HOST_PATH)" \
		docker compose -f DeveloperEdition/launch-docker-compose.yaml pull automation 2>/dev/null || true
	@STAGE0_UMBRELLA_HOST_PATH="$(STAGE0_UMBRELLA_HOST_PATH)" STAGE0_LAUNCHPAD_HOST_PATH="$(STAGE0_LAUNCHPAD_HOST_PATH)" \
		docker compose -f DeveloperEdition/launch-docker-compose.yaml run --rm \
		automation clone-all

delete-services:
	@[ -n "$$GITHUB_TOKEN" ] || (echo "Error: export GITHUB_TOKEN"; exit 1)
	@[ -n "$(SERVICES)" ] || (echo "Error: SERVICES required"; exit 1)
	@STAGE0_UMBRELLA_HOST_PATH="$(STAGE0_UMBRELLA_HOST_PATH)" STAGE0_LAUNCHPAD_HOST_PATH="$(STAGE0_LAUNCHPAD_HOST_PATH)" \
		docker compose -f DeveloperEdition/launch-docker-compose.yaml pull automation 2>/dev/null || true
	@STAGE0_UMBRELLA_HOST_PATH="$(STAGE0_UMBRELLA_HOST_PATH)" STAGE0_LAUNCHPAD_HOST_PATH="$(STAGE0_LAUNCHPAD_HOST_PATH)" \
		docker compose -f DeveloperEdition/launch-docker-compose.yaml run --rm -i \
		-e SERVICES="$(SERVICES)" \
		-e I_CONFIRM_DELETE_SERVICES="$$I_CONFIRM_DELETE_SERVICES" \
		-e I_CONFIRM_SLUG="$$I_CONFIRM_SLUG" \
		automation delete-services

delete-all:
	@[ -n "$$GITHUB_TOKEN" ] || (echo "Error: export GITHUB_TOKEN"; exit 1)
	@STAGE0_UMBRELLA_HOST_PATH="$(STAGE0_UMBRELLA_HOST_PATH)" STAGE0_LAUNCHPAD_HOST_PATH="$(STAGE0_LAUNCHPAD_HOST_PATH)" \
		docker compose -f DeveloperEdition/launch-docker-compose.yaml pull automation 2>/dev/null || true
	@STAGE0_UMBRELLA_HOST_PATH="$(STAGE0_UMBRELLA_HOST_PATH)" STAGE0_LAUNCHPAD_HOST_PATH="$(STAGE0_LAUNCHPAD_HOST_PATH)" \
		docker compose -f DeveloperEdition/launch-docker-compose.yaml run --rm -i \
		-e I_CONFIRM_DELETE_ALL="$$I_CONFIRM_DELETE_ALL" \
		-e I_CONFIRM_SLUG="$$I_CONFIRM_SLUG" \
		automation delete-all