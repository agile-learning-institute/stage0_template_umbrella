# {{info.name}} Developer Edition

The {{info.name}} Developer Edition ``{{info.developer_cli}}`` is a Command Line Interface that provides key components of the developer experience. This CLI wraps docker compose commands, and secret management for local development environments. All developers should install this tooling, create and configure tokens, and review the linked standards before contributing to any repo.

## Step 1 of 4 - Install Prerequisites

Run `make verify` to check that all prerequisites are installed. If any fail, install them using the links below.

### Build tools
- **make** - Usually pre-installed. macOS: Xcode Command Line Tools (`xcode-select --install`). Linux: `apt install build-essential` or equivalent. https://www.gnu.org/software/make/
- **Node.js** (v18+) - https://nodejs.org/en/download
- **npm** (v11.5+) - Bundled with Node.js
- **Vite** - `npm install -g vite` or use via `npx vite`. https://vitejs.dev/guide/

### Python tools
- **Python 3.12+** - https://www.python.org/downloads/
- **Pipenv** - https://pipenv.pypa.io/en/latest/ (`pip install pipenv`)

### Container tools
- **Docker Desktop** - https://www.docker.com/get-started/
- **Docker Buildx** - Bundled with Docker Desktop. Standalone: https://docs.docker.com/buildx/working-with-buildx/

### GitHub & Git
- **GITHUB_TOKEN** - See [Configuring AccessToken](#configure-access-tokens) 
- **gh** (GitHub CLI) - https://cli.github.com/
- **git** - https://git-scm.com/downloads

### Utilities
- **jq** - https://jqlang.github.io/jq/download/ (macOS: `brew install jq`)
- **yq** - https://mikefarah.gitbook.io/yq (macOS: `brew install yq`)
- **curl** - Usually pre-installed. https://curl.se/download.html
- **ssh** - Usually pre-installed. https://www.openssh.com/

### Other
- **zsh shell** - Default on macOS. Linux: https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH
- **Mongo Compass** - https://www.mongodb.com/docs/compass/install/
- **WSL** - For Windows users: https://learn.microsoft.com/en-us/windows/wsl/install

## Step 2 of 4 - Install the CLI
Use these commands to install the Developer Edition ``{{info.developer_cli}}`` command line utility. 
```sh
git clone git@{{org.git_host | replace('https://','') | replace('http://','')}}:{{org.git_org}}/{{info.slug}}.git
cd {{info.slug}}
make install
```

## Step 3 of 4 - Configure access tokens
When local environment values are required (GitHub access tokens, etc.) they are stored in the hidden folder ``~/.{{info.slug}}`` instead of a being replicated across multiple repo level .env files. 

### GITHUB_TOKEN
We are using GitHub to publish the api_utils pypi package, the spa_utils npm package, and GitHub Container Registry to publish containers. Create a GitHub classic access token with `repo` `workflow`, and `write:packages` privileges. Save it as `GITHUB_TOKEN` in the ``~/.{{info.slug}}/`` folder.

To create a token, login to GitHub and click your Profile Pic -> Settings -> Developer Settings -> Personal access tokens -> Tokens(classic) -> Create New -> ✅ repo, ✅ workflow, ✅ write:packages. Users wanting to use Stage0 Delete commands will also need ✅ delete:packages and ✅ delete_repo permissions. For reference: [ghcr and github tokens](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)

### Git HTTPS auth (used by launch scripts)
The launch scripts clone and push via HTTPS using your token. If you normally use SSH and run git steps manually, configure HTTPS auth once:
```sh
export GITHUB_TOKEN=ghp_your_token
git config --global url."https://x-access-token:${GITHUB_TOKEN}@github.com/".insteadOf "https://github.com/"
```
Or clone with the token in the URL: `git clone https://x-access-token:${GITHUB_TOKEN}@github.com/org/repo.git`
  
## Step 4 of 4 - Finally
After you have everything installed and your token in place, run update to finish the install.
```sh
## Update Developer Edition configurations
make update
```

---

## Development Standards
- Understand a few simple [Architecture Principles](./DeveloperEdition/standards/ArchitecturePrinciples.md)
- Review the [Data Standards](./DeveloperEdition/standards/data_standards.md) and install prerequisites.
- Review the [SRE Standards](./DeveloperEdition/standards/sre_standards.md) and install prerequisites.
- Review the [API Standards](./DeveloperEdition/standards/api_standards.md) and install prerequisites.
- Review the [SPA Standards](./DeveloperEdition/standards/spa_standards.md) and install prerequisites.

## Developer Workflow
We utilize an Issue–Feature–Branch pattern for the developer workflow:
- Pick up an issue from the code base and assign yourself. If someone else is assigned to the issue you should check with them before starting any work. 
- Create a branch for the feature you are working on, reference the issue # in the branch name. 
- Commit and push your changes frequently while you are working. 
- When your work is feature complete, and **all unit/integration/blackbox testing** is passing with appropriate coverage, open a pull request (PR) from the feature branch back to main. 

These pull requests must be peer reviewed before being merged back into the main branch of the repository. This review process may require additional updates before it is approved. This "merge to main" event is what drives CI automation. If you are asked to review a PR, do your best to accommodate a prompt review.

If you have questions about implementing a feature, create your feature branch and open a draft PR with detailed questions and request a review of that PR, and then post a link to the PR in the General channel on Discord.

## Umbrella Repo Developer Commands
```sh
# Verify you have all the developer pre-req's installed
make verify

## Install the developer CLI in your search path
make install

## Update the developer CLI with the latest compose file
make update 

## Generate data schemas for all collections in catalog.yaml
make schemas

## Build the welcome page container
make container

```
## Umbrella Repo Stage0 Automation
The command impact multiple repositories
```sh
## Launch all or some services
make launch-all
### or ###
make launch-services "SERVICES=profile sales operations"

## Setup an environment when the Launch is completed
## Will rm -rf any existing repo's (other than the umbrella)
make clean-clone-build

######################################################
######### WARNING - REAL DELETE WITHOUT UNDO #########
######################################################
## Deletes all or some repo's and packages from GitHub and local disk
## NOTE: Does not affect the umbrella repo itself
make delete-all
make delete-services "SERVICES=profile sales operations"
######################################################
######### WARNING - REAL DELETE WITHOUT UNDO #########
######################################################
```
