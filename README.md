# Umbrella Project Template

## Quick Start
This is a template to create the Umbrella Repo for your system. First, create a GitHub organization to host the system. Then in that organization create a new repo using this template. Name your repo with a short one-word product name slug. To avoid complications, make the repo public, and both the organization name and the slug should be all lower case letters. Clone your new repo down to your computer. We suggest cloning it into a ``source`` folder (i.e. ~/source), this is a poly-repo solution, and many repo's will be cloned to this location. 

Now, update the template [.stage0_template/Specifications/product.yaml](./.stage0_template/Specifications/product.yaml) to reflect your use case. See below for pointers on what values to use.
```yaml
info:
  name: <Your Product Name>
  description: <Your Product Description>
  slug: <slug> - That short one-word product name slug.
  developer_cli: <cli> - A very short 2-character CLI command for your product
  db_name: <db_name> Any database name
  base_port: 8383 - A dozen or so port numbers will be assigned starting with this number. 

organization:
  name: <Organization Name>
  email: <info@your-organization.com>
  founded: 2024
  slug: <Github Org Name> i.e. agile-learning-institute
  git_host: https://github.com
  git_org: <same as org slug>
  docker_host: ghcr.io
  docker_org: <same as org slug>
```

Once you have saved those changes use the ``make merge`` command as shown below, and then re-load the README to continue. You will need to have the ``make`` utility and [Docker Desktop](https://www.docker.com/get-started/) installed to merge this template. 

```sh
## Merge your specifications with the template
make merge .stage0_template/Specifications
```

## Contributing
See [Template Guide](https://github.com/agile-learning-institute/stage0_runbook_merge/blob/main/TEMPLATE_GUIDE.md) for information about stage0 merge templates. See the [Processing Instructions](./.stage0_template/process.yaml) for details about this template, and [Test Specifications](./.stage0_template/Specifications/) for sample context data required.

Template Commands
```sh
## Test the Template using test_expected output
## Creates ~/tmp folders 
make test
## Successful output looks like
...
Checking output...
Only in /Users/you/tmp/testRepo: .git
Only in /Users/you/tmp/testRepo/configurator: .DS_Store
Done.

## Look at one file diff from testing
make diff README.md

## Copy a generated file to the test_expected folder
make take somefile.json

## Clean up temp files from testing
## Removes tmp folders
make clean

## Process this merge template using the provided context path
## NOTE: Destructive action, will remove .stage0_template 
## Context path typically ends with ``.Specifications``
make merge {context path}
```
