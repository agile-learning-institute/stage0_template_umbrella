# Stage0 automation: run from this folder (DeveloperEdition/stage0). Specifications at ../../Specifications.
ROOT := $(CURDIR)/../..
SPECS_DIR := $(ROOT)/Specifications
ARCH_FILE ?= $(SPECS_DIR)/architecture.yaml
PRODUCT_FILE ?= $(SPECS_DIR)/product.yaml
SOURCE := $(shell cd $(ROOT)/.. && pwd)
SLUG := $(shell yq -r '.info.slug' $(PRODUCT_FILE))
ORG := $(shell yq -r '.organization.git_org' $(PRODUCT_FILE))
ALL_SERVICES := $(shell yq -r '.architecture.domains[].name' $(ARCH_FILE))

.PHONY: launch-all launch-services clean-clone-build delete-all delete-services validate help

help:
	@echo "Stage0 automation (run from this folder; specifications at ../../Specifications)"
	@echo "  validate                - Verify all prerequisites for launch/delete (build tools, gh, ssh, buildx, git config, Git SSH)"
	@echo "  launch-all              - Build umbrella, publish, launch all services"
	@echo "  launch-services SERVICES=<list> - Launch services (e.g. SERVICES=schema sample)"
	@echo "  clean-clone-build SERVICES=<list> - Clean, clone, build (no publish)"
	@echo "  delete-all              - Delete all service repos and packages"
	@echo "  delete-services SERVICES=<list> - Delete services for given domains"

validate:
	@fail=0; \
	echo "=== Validating prerequisites for launch/delete ==="; \
	echo ""; \
	echo "--- Build tools ---"; \
	command -v make >/dev/null 2>&1 && printf "make:    " && make --version | head -1 || { echo "  FAIL: make"; fail=1; }; \
	command -v node >/dev/null 2>&1 && printf "node:    " && node --version || { echo "  FAIL: node"; fail=1; }; \
	command -v npm >/dev/null 2>&1 && printf "npm:     " && npm --version || { echo "  FAIL: npm"; fail=1; }; \
	(vite --version 2>/dev/null || npx vite --version 2>/dev/null) >/dev/null && printf "vite:    " && (vite --version 2>/dev/null || npx vite --version 2>/dev/null) || { echo "  FAIL: vite"; fail=1; }; \
	echo ""; \
	echo "--- Python tools (3.12) ---"; \
	command -v pipenv >/dev/null 2>&1 && printf "pipenv:  " && pipenv --version || { echo "  FAIL: pipenv"; fail=1; }; \
	PYTEST=$$(mktemp -d); \
	if (cd "$$PYTEST" && pipenv --python 3.12 install >/dev/null 2>&1 && pipenv run python -c "import sys; exit(0 if sys.version_info[:2] == (3, 12) else 1)" >/dev/null 2>&1); then \
		echo "  pipenv+3.12: OK"; \
	else \
		echo "  FAIL: pipenv cannot use Python 3.12"; fail=1; \
	fi; \
	rm -rf "$$PYTEST"; \
	echo ""; \
	echo "--- Container tools ---"; \
	command -v docker >/dev/null 2>&1 && printf "docker:  " && docker --version || { echo "  FAIL: docker"; fail=1; }; \
	docker buildx version >/dev/null 2>&1 && printf "buildx:  " && docker buildx version || { echo "  FAIL: docker buildx"; fail=1; }; \
	echo ""; \
	echo "--- GitHub & Git ---"; \
	[ -n "$${GITHUB_TOKEN:-}" ] && printf "GITHUB_TOKEN: set\n" || { echo "  FAIL: GITHUB_TOKEN (set env var)"; fail=1; }; \
	command -v gh >/dev/null 2>&1 && printf "gh:      " && gh --version | head -1 || { echo "  FAIL: gh"; fail=1; }; \
	command -v git >/dev/null 2>&1 && printf "git:     " && git --version || { echo "  FAIL: git"; fail=1; }; \
	if ! git config --global user.name >/dev/null 2>&1; then echo "  FAIL: git config --global user.name"; fail=1; fi; \
	if ! git config --global user.email >/dev/null 2>&1; then echo "  FAIL: git config --global user.email"; fail=1; fi; \
	echo ""; \
	echo "--- Utilities ---"; \
	command -v jq >/dev/null 2>&1 && printf "jq:      " && jq --version || { echo "  FAIL: jq"; fail=1; }; \
	command -v yq >/dev/null 2>&1 && printf "yq:      " && yq --version || { echo "  FAIL: yq"; fail=1; }; \
	command -v curl >/dev/null 2>&1 && printf "curl:    " && curl --version | head -1 || { echo "  FAIL: curl"; fail=1; }; \
	command -v ssh >/dev/null 2>&1 && printf "ssh:     " && ssh -V 2>&1 || { echo "  FAIL: ssh"; fail=1; }; \
	echo ""; \
	echo "--- Git SSH (clone + push) ---"; \
	GIT_SSH_TEST=$$(mktemp -d); \
	if (git clone git@github.com:agile-learning-institute/testing.git "$$GIT_SSH_TEST" >/dev/null 2>&1 && \
		cd "$$GIT_SSH_TEST" && \
		echo " " >> README.md && \
		git add README.md && \
		git commit -m "validate: SSH clone/push test" >/dev/null 2>&1 && \
		git push >/dev/null 2>&1); then \
		echo "  Git SSH: OK"; \
	else \
		echo "  FAIL: Git SSH clone or push (check SSH key, push access to agile-learning-institute/testing)"; \
		fail=1; \
	fi; \
	rm -rf "$$GIT_SSH_TEST"; \
	echo ""; \
	if [ $$fail -eq 1 ]; then \
		echo "Some prerequisites are missing. See README.md in this folder."; \
		exit 1; \
	fi; \
	echo "=== All prerequisites validated ==="

launch-all:
	@START=$$(date +%s); \
	$(MAKE) -C $(ROOT) build-package publish-package && $(MAKE) launch-services SERVICES="$(ALL_SERVICES)"; \
	END=$$(date +%s); \
	echo "Launch Completed - Started at $$START To $$END Duration: $$((END - START)) Seconds"

launch-services:
	@if [ -z "$(SERVICES)" ]; then echo "Error: SERVICES required. e.g. make launch-services SERVICES=profile"; exit 1; fi; \
	[ -z "$${GITHUB_TOKEN:-}" ] && { echo "Error: GITHUB_TOKEN env required"; exit 1; }; \
	echo "Configuring git and docker for push..."; \
	echo "$$GITHUB_TOKEN" | docker login ghcr.io -u $(ORG) --password-stdin && \
	git config --global --unset-all url."https://@github.com/".insteadOf 2>/dev/null || true && \
	git config --global url."https://x-access-token:$$GITHUB_TOKEN@github.com/".insteadOf "https://github.com/" && \
	for svc in $(SERVICES); do \
		[ -z "$$svc" ] && continue; \
		REPO_LINES=$$(yq -r '.architecture.domains[] | select(.name == "'$$svc'") | .repos[] | select(.type == "api" or .type == "spa") | (.name + "|" + .template + "|" + (.publish // ""))' $(ARCH_FILE) 2>/dev/null) || true; \
		[ -z "$$REPO_LINES" ] && continue; \
		echo "--- Domain: $$svc ---"; \
		echo "$$REPO_LINES" | while IFS='|' read -r repo_name template publish; do \
			[ -z "$$repo_name" ] && continue; \
			REPO_FULL="$(SLUG)_$$repo_name"; \
			REPO="$(ORG)/$$REPO_FULL"; \
			echo ""; \
			echo "##############################################################################"; \
			echo "###   Starting Repo $$REPO_FULL"; \
			echo "##############################################################################"; \
			echo "  Creating $$REPO from template $$template"; \
			rm -rf "$(SOURCE)/$$REPO_FULL"; \
			cd "$(SOURCE)" && gh repo create "$$REPO" --template "$$template" --public || { echo "Failed: $$REPO"; exit 1; }; \
			echo "  Waiting for repo to be ready..."; sleep 5; \
			echo "  Cloning $$REPO"; \
			cd "$(SOURCE)" && git clone "https://x-access-token:$$GITHUB_TOKEN@github.com/$$REPO.git" "$$REPO_FULL" || { echo "Failed clone: $$REPO"; exit 1; }; \
			echo "  Merging $$REPO_FULL"; \
			cd "$(SOURCE)/$$REPO_FULL" && SERVICE_NAME=$$svc make merge "$(SPECS_DIR)" || { echo "Failed merge: $$REPO_FULL"; exit 1; }; \
			if [ -n "$$publish" ]; then \
				echo "  Build-package & publish-package $$REPO_FULL ($$publish)"; \
				case "$$publish" in \
					make)   make build-package && make publish-package ;; \
					npm)    npm run build-package && npm run publish-package ;; \
					pipenv) pipenv run build-package && pipenv run publish-package ;; \
					*)      echo "Unknown publish: $$publish"; exit 1 ;; \
				esac || { echo "Failed build: $$REPO_FULL"; exit 1; }; \
			fi; \
			echo "  Commit & push $$REPO_FULL"; \
			git add -A && git commit -m "Template Merge Processing Complete" && git push origin main || { echo "Failed push: $$REPO_FULL"; exit 1; }; \
			echo ""; \
			echo "##############################################################################"; \
			echo "###   Repo $$REPO_FULL Shipped"; \
			echo "##############################################################################"; \
			cd "$(ROOT)" || true; \
		done; \
	done
	@echo "Launch complete."

clean-clone-build:
	@if [ -z "$(SERVICES)" ]; then echo "Error: SERVICES required. e.g. make clean-clone-build SERVICES=profile"; exit 1; fi; \
	for svc in $(SERVICES); do \
		[ -z "$$svc" ] && continue; \
		REPO_LINES=$$(yq -r '.architecture.domains[] | select(.name == "'$$svc'") | .repos[] | select(.type == "api" or .type == "spa") | (.name + "|" + (.publish // ""))' $(ARCH_FILE) 2>/dev/null) || true; \
		[ -z "$$REPO_LINES" ] && continue; \
		echo "--- Domain: $$svc ---"; \
		echo "$$REPO_LINES" | while IFS='|' read -r repo_name publish; do \
			[ -z "$$repo_name" ] && continue; \
			REPO_FULL="$(SLUG)_$$repo_name"; \
			REPO="$(ORG)/$$REPO_FULL"; \
			echo "  Clean $$REPO_FULL"; \
			rm -rf "$(SOURCE)/$$REPO_FULL"; \
			echo "  Clone $$REPO"; \
			cd "$(SOURCE)" && git clone "https://github.com/$$REPO.git" "$$REPO_FULL" || { echo "Failed clone: $$REPO"; exit 1; }; \
			echo "  Build $$REPO_FULL ($$publish)"; \
			cd "$(SOURCE)/$$REPO_FULL" && ( \
				case "$$publish" in \
					make)   make build-package ;; \
					npm)    npm run build-package ;; \
					pipenv) pipenv run build-package ;; \
					*)      echo "Skipped (no publish): $$REPO_FULL" ;; \
				esac \
			) || true; \
			cd "$(ROOT)" || true; \
		done; \
	done
	@echo "Clean-clone-build complete."

delete-all:
	@START=$$(date +%s); \
	$(MAKE) delete-services SERVICES="$(ALL_SERVICES)"; \
	END=$$(date +%s); \
	echo "Delete Completed - Started at $$START To $$END Duration: $$((END - START)) Seconds"

delete-services:
	@echo "WARNING: This will DELETE repos and packages. This action is NOT reversible."; \
	read -p "Proceed? (y/n) " confirm && [ "$$confirm" = "y" ] || { echo "Aborted."; exit 1; }; \
	for svc in $(SERVICES); do \
		[ -z "$$svc" ] && continue; \
		REPO_LINES=$$(yq -r '.architecture.domains[] | select(.name == "'$$svc'") | .repos[] | select(.type == "api" or .type == "spa") | (.name + "|" + (.publish // ""))' $(ARCH_FILE) 2>/dev/null) || true; \
		[ -z "$$REPO_LINES" ] && continue; \
		echo "--- Domain: $$svc ---"; \
		echo "$$REPO_LINES" | while IFS='|' read -r repo_name publish; do \
			[ -z "$$repo_name" ] && continue; \
			REPO_FULL="$(SLUG)_$$repo_name"; \
			REPO="$(ORG)/$$REPO_FULL"; \
			if [ -d "$(SOURCE)/$$REPO_FULL" ] && [ -n "$$publish" ]; then \
				echo "  Delete-package $$REPO_FULL ($$publish)"; \
				cd "$(SOURCE)/$$REPO_FULL" && ( \
					case "$$publish" in \
						make)   make delete-package ;; \
						npm)    npm run delete-package ;; \
						pipenv) pipenv run delete-package ;; \
						*)      ;; \
					esac \
				) 2>/dev/null || true; \
			fi; \
			echo "  Delete repo $$REPO"; \
			gh repo delete "$$REPO" --yes 2>/dev/null || echo "  (repo may not exist)"; \
			echo "  Clean $$REPO_FULL"; \
			rm -rf "$(SOURCE)/$$REPO_FULL"; \
			cd "$(ROOT)" || true; \
		done; \
	done; \
	echo "Delete complete."