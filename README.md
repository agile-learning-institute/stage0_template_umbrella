# Umbrella Project Template

## Quick Start
This is a template to create the Umbrella Repo for your system. Create a new repo using this template and clone it down to your computer. We suggest cloning it into a ``source`` folder, many repo's will be cloned to this location. 

Now, update the template [.stage0_template/Specifications/product.yaml](./.stage0_template/Specifications/product.yaml) to reflect your use case. See below for pointers on what values to use.
```yaml
info:
  name: <Your Product Name>
  description: <Your Product Description>
  slug: <slug> - A short one-word name for your product
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
