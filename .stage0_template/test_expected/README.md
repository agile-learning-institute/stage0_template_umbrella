# Agile Learning Institute Mentor Hub

## Stage0 Demo Quickstart
Use the instructions below to 
- Familiarize yourself with project [Standards](#familiarize-yourself-with-standards)
- Update your [product Specifications](#update-your-specifications) 
- [Run an AI Task](#use-ai-task-to-propagate-changes) to propagate changes.
- Install the [Developer CLI](#install-the-developer-cli) and prerequisites
- [Launch](#launch-your-product) 🚀 your product!

--- 

### Familiarize yourself with standards. 
- Review [Architecture Principles](./DeveloperEdition/standards/ArchitecturePrinciples.md) to understand the architecture supported by the current templates.
- Review the various [Coding Standards](./DeveloperEdition/standards/) implemented in the API, UI, DB, and SRE domains.
- Review the [Specifications](./Specifications/) to understand the starting point.

Note: Now is the time for your team to review and approve or modify these standards. The tooling will work with alternative technologies or architectures. Any updates will likely require template refactors, but the effort to do so is minimal for most changes.

---

### Update your Specifications
Design specifications are documented as ``yaml`` files in the Specifications folder. The three key files used by the templates are:
- [product.yaml](./Specifications/product.yaml) that you created when you merged the template.
- [catalog.yaml](./Specifications/catalog.yaml) that will define the top-level data domains within your system. 
- [architecture.yaml](./Specifications/architecture.yaml) that describes the micro-service domains of your system. 

Start by updating your [catalog.yaml](./Specifications/catalog.yaml) to list the data domains of your system. These will align with MongoDB Collections in the generated code.This is an example used by the Mentor Hub testing project.
```yaml
data_dictionaries:
  - name: Identity
    description: The Identity of a person who is a user of the CreatorDashboard system. 
  - name: Profile
    description: User Profile associated with one or more **Users**, and optionally with an **Identity**.
  - name: Platform
    description: A social media platform which hosts **Users** and **Posts**.
  - name: User
    description: A User of a social media platform. Some **Users** are **Creators**.
  - name: Dashboard
    description: A Dashboard Configuration for a **Profile**
  - name: Post
    description: The content published by a **Creator** on a **Platform**. Post types can be Video, Image, or Text.
  - name: Comment
    description: A comment, created by a **User**, on a **Post**. A Comment can have **Comments**
  - name: Sentiment
    description: A sentiment of a **Post** or **Comment**.
  - name: Classification
    description: A classification of a **Post** or **Comment**.
  - name: TestRun
    description: A LLM Model/Prompt test run - generates **Grades** from **TestData**
  - name: TestData
    description: A collection of data used to test a **TestRun**.
  - name: Grade
    description: A LLM Model/Prompt test run - generates **Grades** from **TestData**```

Now update your architecture.yaml file to describe the micro-services in your system. The current file contains a sample microservice 
```yaml
    - name: sample
      description: Sample Service Domain - use for each User Journey domain.
      data_domains:
        controls:
          - Control 
        creates:
          - Create 
        consumes:
          - Consume
      repos:
        - name: sample_api
          description: Flask API with MongoDB Database
          template: agile-learning-institute/stage0_template_flask_mongo
          publish: pipenv
          type: api
          port: 8389
        - name: sample_spa
          description: Vue Vuetify SPA
          template: agile-learning-institute/stage0_template_vue_vuetify
          publish: npm
          type: spa
          port: 8390
```
Replace this service with your own. This is an example of the Creator Dashboard testing data, 
note that all of the data_domains listed are present in the catalog.yaml file. 
This is critical and misalignment will result in non-functional code.
```yaml
    - name: profile
      description: User Profile Management.
      data_domains:
        controls:
          - Profile
          - Platform
          - User
        creates: []
        consumes:
          - Identity
      repos:
        - name: profile_api
          description: Consumes Identity Service Bus events and manages Profile data.
          template: agile-learning-institute/stage0_template_flask_mongo
          publish: pipenv
          type: api
          port: 9096
        - name: profile_spa
          description: Profile API Status Single Page Web Application
          template: agile-learning-institute/stage0_template_vue_vuetify
          publish: npm
          type: spa
          port: 9097
    - name: evaluator
      description: Sentiment Analysis Prompt and Model evaluation tool
      data_domains:
        controls:
          - TestRun
          - TestData
        creates:
          - Grade
        consumes:
          - Profile
      repos:
        - name: evaluator_api
          description: Evaluator API Server
          template: agile-learning-institute/stage0_template_flask_mongo
          publish: pipenv
          type: api
          port: 9098
        - name: evaluator_spa
          description: Evaluator Single Page Web Application - setup/schedule/run/view test results
          template: agile-learning-institute/stage0_template_vue_vuetify
          publish: npm
          type: spa
          port: 9099
    - name: dashboard
      description: YouTube Ratio Reporting - Product MVP.
      data_domains:
        controls:
          - Dashboard
        creates:
          - Post
          - Comment
        consumes:
          - Classification
          - Profile
      repos:
        - name: dashboard_api
          description: Dashboard API Server
          template: agile-learning-institute/stage0_template_flask_mongo
          publish: pipenv
          type: api
          port: 9100
        - name: dashboard_spa
          description: Dashboard Single Page Web Application
          template: agile-learning-institute/stage0_template_vue_vuetify
          publish: npm
          type: spa
          port: 9101
    - name: classifier
      description: Comment Sentiment Analysis and Classifier 
      data_domains:
        controls:
          - sentiment
          - ratios
        creates: []
        consumes:
          - post
          - comment
          - user
      repos:
        - name: classifier_api
          description: Sentiment Analysis Classifier API Server
          template: agile-learning-institute/stage0_template_flask_mongo
          publish: pipenv
          type: api
          port: 9102
        - name: classifier_spa
          description: Sentiment Analysis Classifier Single Page Web Application
          template: agile-learning-institute/stage0_template_vue_vuetify
          publish: npm
          type: spa
          port: 9103
```

### Use AI Task to propagate changes. 

Now is a good time to make a commit! 
Copy - paste this prompt to your AI Code Assistant:

---

Review this [README](./Tasks/README.md) for context and execute [this task](./Tasks/AS_NEEDED.R100.after_specs_update.md).

---

### Install the Developer CLI
Now is a good time for a commit! 
Install the developer CLI and all of the developer pre-req's so that the Launch scripts can run successfully. 
Follow all of the instructions in [CONTRIBUTING.md](./CONTRIBUTING.md)
```sh
make install
cp <your token> ~/.mentorhub/GITHUB_TOKEN
make update
source ~/.zshrc
```

### Launch your product.
Use these commands to generate, clone, merge, and build all of your repo's and then start the containers in your local dev environment.
```sh
make launch-all
mh up all
```

The ``launch-all`` command may take a while to run, more than 10 minutes. After using the ``mh up all`` command visit the [welcome page](localhost:8080) in your browser.

Now is a good time to add all of the generated repo's to your IDE workspace (And delete this part of the README). At some point the SRE's may want to remove some of the scripts in the Makefile. Commands like ``delete-all``, ``clean-clone-build``, and ``launch-all`` should have a limited lifetime in the repo. Keep them around until you are sure that the architecture has settled and no more code generation is needed. 


---

## Big Idea
A platform to connect mentors with engineers engaged in a life long learning journey.

## Development Team 
- 
- 

## Design Specifications
- [Product Description](./Specifications/product.yaml) 
- [Stakeholders](./Specifications/stakeholders.yaml)
- [Product Roadmap](./Specifications/roadmap.yaml)
- [Data Catalog](./Specifications/catalog.yaml)
- [Architecture Diagram](./Specifications/architecture_diagram.md)
- [Architecture Data](./Specifications/architecture.yaml)

## Contributing Guides
- [Developer Onboarding](./CONTRIBUTING.md) On-Boarding Process and CLI install
- [Architecture Principles](./DeveloperEdition/standards/ArchitecturePrinciples.md)
- [Data Standards](./DeveloperEdition/standards/data_standards.md)
- [API Standards](./DeveloperEdition/standards/api_standards.md)
- [UI Standards](./DeveloperEdition/standards/spa_standards.md)
- [SRE Standards](./DeveloperEdition/standards/sre_standards.md)