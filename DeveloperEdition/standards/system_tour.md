# System Tour

Welcome to the {{info.name}} System - a poly-repo, backend for frontend, microservice architecture application. This is a tour-guide that will help you familiarize yourself with the codebase. Don't feel too overwhelmed, you will only work on a single repo at a time, this tour will introduce you to all of them. 

## Developer Edition
If you haven't already installed the Developer Edition cli ``{{info.developer_cli}}`` you should do so now. Then let's take a look at what that tool actually does. 

``{{info.developer_cli}}`` is just a bash script - that largely just wraps some ``docker compose`` commands. You should browse that script, and the [docker-compose.yaml](../docker-compose.yaml) to understand how the application is divided into services, and how they are run altogether or one at a time. 

Now - let's use ``{{info.developer_cli}}`` to pull the latest containers from our GitHub Container Registry. Use ``{{info.developer_cli}} pull all`` - this command will pull all of the latest containers from GitHub for use with ``{{info.developer_cli}} up`` commands. When you are working on a repo - you will re-build that container and test it locally. Multi-architecture (ARM & AMD) containers are built by CI (Continuous Integration) that is implemented as GitHub Action's that run when ever a feature branch is merged back to the main branch. 

Now - let's use ``{{info.developer_cli}}`` to start the whole system. Use ``{{info.developer_cli}} up all`` and then visit [localhost:8080](http://localhost:8080/) and explore the system. When you are done exploring use ``{{info.developer_cli}} down`` to stop all the containers. 

## Clone Everything
Ok - now to look at all of the different repo's. You will first need to clone them down to your computer - you can do so using the stage0 launch UI. Use these commands:
- ``{{info.developer_cli}} down``       # to shut down the app if it's running
- ``make stage0-launch-ui``         # to start the launch tools

Now open your browser to [localhost:8080](http://localhost:8080/) and click the "all" checkbox, and click the "Clone" button. When you are done, click **Exit** and the launch utility container will stop automatically.

## Tour common code libraries
With the Backend for Frontend pattern, all of our services consist of a single API that supports a single SPA. Common code that is used by multiple API's or SPA's is shared in utility repo's. Review these repo's to see the overall patterns used.

### {{info.slug}}_api_utils
Review the README - and then try these developer commands. 
- ``pipenv install --dev``  # to install dependencies
- ``pipenv run test``       # to run unit testing     
- ``pipenv run db``         # to run a backing MongoDB database
- ``pipenv run dev``        # to run the API Demo Server (requires a db, captures command line)
- ``pipenv run e2e``        # to run black-box e2e testing (requires dev mode server)

Leave the dev server running while we move on to the SPA

### {{info.slug}}_spa_utils
Review the README - and then try these developer commands
- ``npm install --include=dev`` # to install dependencies
- ``npm run test``          # to run unit testing
- ``npm run dev``           # to run UI Demo Server (captures command line, requires API dev server)
- ``npm run cypress:run``   # to run black-box cypress tests (requires SPA dev server)
You can now stop the API and SPA dev servers before moving on

## Tour {{info.slug}} Services

Each service in the system has a paired API and SPA. Review the README's in each Repo (they all rhyme) and then try these developer commands - they should work in every API/SPA repo.

### {{info.slug}}_*_api
- ``pipenv install --dev``
- ``pipenv run test``       # to run unit testing
- ``pipenv run db``         # to start a backing MongoDB Database
- ``pipenv run dev``        # to start the API Server in dev mode (captures command line)
- ``pipenv run e2e``        # to run black-box e2e testing (requires API running)
- ``pipenv run container``  # to to build a container 
- ``pipenv run api``        # to run the API container (and backing database)

### {{info.slug}}_*_spa
- ``npm install --include=dev``
- ``npm run test``          # to run unit testing
- ``npm run api``           # to run the backing API and Database
- ``npm run dev``           # to run the UI in Dev Mode (captures command line)
- ``npm run cypress:run``   # to run the Cypress tests (requires UI)
- ``npm run container``     # to build a container
- ``npm run service``       # to run the Database + API + SPA containers

## Schema editor
As we work on this system we will be using the Schema Configurator tool to describe data structures, generate JSON Schema for use with Task Automation, and configure MongoDB for use with the system. From the launchpad, ``cd {{info.slug}}_mongodb_api``, and use ``make dev`` from that folder to start the Configurator in Edit mode. Click the ? icon and review the help screens. 

## Task Automation 
Every repo has a /Tasks folder, with a README that describes the Task Automation framework. This framework is used to create re-usable LLM tasks for working in a repo. You can review existing tasks, or ask your AI Code Assistant to review the Task Automation README.md and help you create a new task for something you want to accomplish. 

## Merge Templates and launch automation
**Extra Credit** If you want to understand the tooling that was used to help launch this product, you can review the README at [stage0_launch](https://github.com/agile-learning-institute/stage0_launch) which uses [stage0_runbook_merge](https://github.com/agile-learning-institute/stage0_runbook_merge) to automate repository provisioning.