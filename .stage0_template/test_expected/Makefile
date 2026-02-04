.PHONY: help install update spark-runbook schemas container

help:
	@echo "Mentor Hub Developer CLI - Available commands:"
	@echo ""
	@echo "  make install        - Install MentorHub CLI tools to ~/.MentorHub"
	@echo "  make update         - Update MentorHub CLI tools and configure Docker/Git"
	@echo "  make schemas        - Fetch JSON schemas for all data dictionaries, assumes mongodb_api is running"
	@echo "  make container      - Build the Mentor Hub welcome page Docker container locally"
	@echo ""
	@echo "For more information, see ./DeveloperEdition/README.md"

install:
	@echo "Installing MentorHub CLI..."
	@mkdir -p ~/.MentorHub
	@if ! grep -q "Added by MentorHub CLI install" ~/.zshrc 2>/dev/null; then \
		echo "\n# Added by MentorHub CLI install" >> ~/.zshrc; \
		echo "export PATH=\$$PATH:~/.MentorHub" >> ~/.zshrc; \
		echo "export GITHUB_TOKEN=\$$(cat ~/.MentorHub/GITHUB_TOKEN)" >> ~/.zshrc; \
		echo "Added ~/.MentorHub to PATH in ~/.zshrc"; \
	else \
		echo "~/.MentorHub already in PATH"; \
	fi
	@echo "Installation complete. Run 'source ~/.zshrc' or restart your terminal."

update:
	@echo "Updating MentorHub CLI..."
	@if [ ! -f ~/.MentorHub/GITHUB_TOKEN ]; then \
		echo "Error: GITHUB_TOKEN not found! - See ./DeveloperEdition/README.md"; \
		exit 1; \
	fi
	@cp ./DeveloperEdition/mh ~/.MentorHub/mh && \
	chmod +x ~/.MentorHub/mh && \
	cp ./DeveloperEdition/docker-compose.yaml ~/.MentorHub/docker-compose.yaml && \
	GITHUB_TOKEN=$$(cat ~/.MentorHub/GITHUB_TOKEN) && \
	echo "$$GITHUB_TOKEN" | docker login ghcr.io -u agile-crafts-people --password-stdin && \
	echo "Docker login completed" && \
	git config --global --unset-all url."https://@github.com/".insteadOf 2>/dev/null || true && \
	git config --global url."https://$$GITHUB_TOKEN@github.com/".insteadOf "https://github.com/" && \
	echo "Git URL configured" && \
	echo "Updates completed"

schemas:
	@echo "Fetching JSON schemas for all data dictionaries..."
	@mkdir -p ./Specifications/schemas
	@yq eval '.data_dictionaries[] | "\(.name)|\(.version)"' ./Specifications/catalog.yaml | \
	while IFS='|' read -r name version; do \
		echo "Fetching schema for $${name} (version $${version})..."; \
		curl -s "localhost:8180/api/configurations/json_schema/$${name}.yaml/$${version}" > "./Specifications/schemas/$${name}.schema.json" \
		|| echo "Warning: Failed to fetch schema for $${name}"; \
	done
	@echo "Schema fetching complete."

container:
	@echo "Building Creator Dashboard container..."
	@docker build -t ghcr.io/agile-learning-institute/MentorHub:latest .
	@echo "Container built successfully: ghcr.io/agile-learning-institute/MentorHub:latest"