# Terraform general

## Prerequisites

Read the [Terraform general](terraform_general.md) contributing guide, and set up your development environment.

## Creating repository

Use [terraform-module-template](https://github.com/opsd-io/terraform-module-template) to create new Terraform module.
The name of the repository should match one of formats:

* `terraform-module-$provider-$resource` - for basic modules that wraps provider resource with or without additional resources.
  Additional resources must be part of main resource, owned exclusively, with strong affiliation to it. Good examples for composite modules are: virtual machine with additional disks, virtual network with sub-networks or database cluster with multiple nodes.

* `terraform-module-$provider-$functionality` - for more complex aggregation of resources that solves particular use case.
  This type of modules comes with limited configuration of resources to fit specific situation, even if resource itself is capable of more.
  These modules may, but do not have to, use generic modules, and the presence of ones does not exclude the existence of others.
  An example would be *terraform-module-aws-static-website* that combines CloudFront CDN and S3 buckets to serve static files, or *terraform-module-aws-backend* that combines S3 bucket with DynamoDB to provide Terraform state backend storage.
  Even if both mentioned modules use S3, they are doing it in different way, requiring different configuration options. Still, generic *terraform-module-aws-s3-bucket* might exists, with most complex set of configuration options.

Make sure the repository settings are set: pull request options and branch protection rules.
<!-- TODO: write more about it. -->

## Repository structure

**WARNING**: current template structure is bit off. Update in progress..

The OPSd module repository structure is based on [Standard Module Structure](https://developer.hashicorp.com/terraform/language/modules/develop/structure).
Most important aspects are described below:
* `LICENSE` - license under which this module is available.
* `README.md` - up to date description of the module, what it should be used for and how. This file should is updated by *terraform-docs* tool.
* `variables.tf` - all module variables must be in this file. If there are none, file should be removed.
* `main.tf` - this file should contain all data sources, resources, module calls and local variables.
  If this file becomes too big to work with or resources could be grouped together it should be spitted to multiple `*.tf` files - see rules below.
* `outputs.tf` - all module outputs must be in this file. If there are none, file should be removed.
* `modules/` - directory for nested modules. They must obey the same rules as main module.
* `examples/` - directory for examples of using this module. Each example should be placed in separate subdirectory.
  The example directory should have README file that explains what this example is about.

In addition to the terraform files, there are configuration files in the repository.
These files are only for developers and are not important for the operation of the module:

* `.envrc` - [direnv](https://direnv.net/) configuration.
* `.github/CODEOWNERS` - define who is responsible for code in a repository.
* `.github/workflows/` - [GitHub Actions](https://docs.github.com/en/actions/quickstart), workflows (aka. CI/CD pipelines).
* `.pre-commit-config.yaml` - [pre-commit]() configuration.
* `.terraform-docs.yml` - [terraform-docs]() configuration.
* `.tflint.hcl` - [tflint](https://github.com/terraform-linters/tflint) configuration.
* `.tool-versions` - [asdf](https://asdf-vm.com/) configuration.


## Terraform files

Terraform files must use UTF-8 encoding without BOM, Unix-line line endings (LF).
Indent with two spaces, not tabs.
No whitespaces at line endings.
Newline at end of file is mandatory.

Remember to `terraform fmt` - before committing your code.
It is also adviced to check code syntax and conventions with `tflint`.
Both tools are automatically called by `pre-commit` hooks and their status is also validated in pull request checks.
**Notice**: some rules that are listed here, are not enforced by syntax checking tools, you have to verify this on your own manually.

## Naming objects

Every variable, resource, module invocation and output must use snake case - only lower case, each space is replaced by an underscore (`_`) character.
The names of variables and outputs as a interface of module must be descriptive and straightforward to understand.
The names of resources, as an internal thing might be less clear, as long as they stay meaningful.
For more information, read more about variables, outputs and resources.

## Comments

Comment should start with single `#` sign, followed by single space.
Usage of `//` is not allowed.
Multiline comments might be delimited by `/*` and `*/`, but it's preferable to use multiple `#` lines.

Avoid fancy-formatting, ASCII-art boxes, multiple `#` signs at beginning or any at end of lines, etc.

It is allowed to add single or multiline comment at beginning of a file to explain it's purpose, but only if it adds additional information.
This comment must be followed by single empty line.

It is also accepted to add single line comment, surrounded by empty lines, that describes resources bellow, creating something like a group.

Comment describing particular resource should be right before it, without empty line between comment and resource block.

Avoid unnecessary comments, that means comments that does not provide any additional information.
For example the `aws_instance.bastion` comment should NOT be "The AWS instance for bastion host" - the resource type and name already explained this.

In contrast to meaningless comments, some resources or expressions might require explanation.
This is true for complex firewall rules, policies, or resources which purpose isn't clear in context of module.
Best example as multi-line expressions, especially for-expressions (aka. list/map comprehension) that transforms one structure to another.
It is highly advised to explain why and how it is made.

**Rule of thumb**: less is better. Add comments only if it might help someone else to understand your code.

Example file with lots of comments:

```hcl
# This file is pure abstract example.
# All resources here have no meaning, the code itself will not work!

resource "aws_vpc" "main" {
  # arguments..
}

resource "aws_s3_bucket" "images" {
  # arguments..
}

# Bucket for ElasticSearch snapshots at managed RDS.
resource "aws_s3_bucket" "es_backups" {
  # arguments..
}

# VPC routing tables & rules for current environment.

resource "aws_route_table" "public" {
  # arguments..
}

resource "aws_route_table" "private" {
  # arguments..
}

resource "aws_route" "site_a" {
  # arguments..
}

resource "aws_route" "site_b" {
  # arguments..
}

resource "aws_route" "mgmt_peering" {
  # arguments..
}
```
The file starts with two lines of comment describing it's purpose.
Both `aws_vpc.main` and `aws_s3_bucket.images` don't have a comment, as it's clear what they are for.
The name of `aws_s3_bucket.es_backups` seems obvious, but comment gives additional context about how (snapshots) and where (RDS) this bucket is used for backup.
Next comment separates previous resources from routing tables and their rules. If there were more of them, they might be moved to separate file ie. `routing.tf`, but this is not required.

Please notice the names of resources too:
There is just one VPC managed here so "main" is good name.
Each S3 bucket resource have short but meaningful name.
The route tables names are simple as "public" and "private" to avoid code-smells like `aws_route_table.private_route_table` or blank names like "rt1" and "rt2".

## Terraform block

Terraform settings are gathered in the `terraform` block.
For modules itself, this block should be at the beginning of `main.tf` file.
Version constrains should define only their minimum allowed version, unless it is know it will not work with particular version (due to bugs) or too new (because of known incompatibilities).

First field in block must be the `required_version` to define Terraform runtime version, then after empty line the `required_providers` block with all providers required by this module defined in alphabetical order.
No empty lines are allowed inside required providers block.

Example `main.tf` file:
```hcl
terraform {
  required_version = ">= 1.5.5"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.12"
    }
    github = {
      source  = "integrations/github"
      version = ">= 6.0.1"
    }
  }
}

# resources goes here
```

### Root modules

For root-modules, including examples, the `terraform` block should be in separate file `terraform.tf`, together with providers configuration (all `provider` blocks).
Providers blocks must be after Terraform settings, with empty lines between them, defined in alphabetical order.
Providers that require no configuration (e.g. random or archive) or when all arguments are specified using environment variables should not have any blocks, unless aliases are needed.

Example `terraform.tf` file:
```hcl
terraform {
  # same as above
}

provider "aws" {
  # configuration options
}

provider "github" {
  # configuration options
}
```

### Backend configuration

Despite the `backend` block is nested within `terraform` block, its definition must be in separate `backend.tf` file with all common settings like backend type and fixed configuration options.
The distinctions for each root-module (ie. different path name) must be set in `backend-config.tfvars` file.
Reasoning for this is much easier synchronization (copy-paste or symlink friendly) of this block among multiple root modules.

Module examples must not define a backend, silently using the default `local` backend, which stores state in current working directory.

## Input variables

All input variables must be in `variables.tf` file.
Each variable must have valid name (see naming rules), description and type.
If input value is sensitive, `sensitive = true` is required, explicit false value is optional, but advised if any other inputs are sensitive.
If input value cannot be null, the `nullable = false` should be used, explicit true value is optional.

Using defaults is restricted to:
* if variable is passed directly to resource into optional parameter, then `default = null` can be used.
* if variable type is `list(..)` or `set(..)`, and empty value is allowed, then `default = []` can be used.
* if variable type is `set(..)`, and empty value is allowed, then `default = {}` can be used.
* if value can have any "good practice" or "safe" default value it should be used.
  For example default port for service or smallest reasonable OS-disk size.

Input variables blocks must be separated by single empty line or double empty line to group them, if needed.

**Order of arguments**: description, type, sensitive, nullable, default.
**Reason**: description is close to name; when default is last, multi-lined expressions are more readable.
The sensitive and nullable flags are in the middle as the only remaining option.
The optional validation blocks are at the end, with empty lines before each block.

The complete variable sample:
```hcl
variable "name" {
  description = "The name for the resource."
  type        = string
  sensitive   = false
  nullable    = true
  default     = "foobar"

  validation {
    condition     = length(var.name) > 2
    error_message = "The name must be at least 3 chars long."
  }

  validation {
    condition     = length(var.name) < 64
    error_message = "The name must be at less than 64 chars long."
  }
}
```

### Naming rules

Whenever module managed single resource, variables prefixes should be avoided.
For example if module manages "foobar" resource, it's better to have `var.name` and `var.size` rather than `var.foobar_name` and `var.foobar_size` as long as it's straightforward.

When module manages few resources that are strongly associated ie. virtual machine, it's os-disk and network card, then `var.name` is still good for VM name, because it is the primary resource that module manages.
For os-disk and network card as supplementary resources, the `var.os_disk_name` and `var.nic_name` should be used.
This rule works when one resources is "parent" and others are "children" that cannot exists independently.

When module manages few resources that are weakly associated, or it is hard to point the "root" resource, it is advised to prefix both variables to avoid confusion.
For example module that creates compute cluster and load-balancer, the `var.name` is vague and should be avoided.
In this situation the `var.cluster_name` and `var.lb_name` should be used.

## Output values

All module outputs must be in `outputs.tf` file.
Each output must have valid name (see naming rules) and description.
If output value is sensitive, `sensitive = true` is required, explicit false value is optional, but advised if any other outputs are sensitive.

No empty lines inside output block is allowed.
Output blocks must be separated by single empty line or double empty line to group outputs.

**Order of arguments**: description, sensitive, value.
**Reason**: description is close to name; when value is last, multi-lined expressions are more readable.
The sensitive flag is in the middle as the only remaining option.

The complete output sample:
```hcl
output "id" {
  description = "The ID of null_resource instance."
  sensitive   = false
  value       = null_resource.test.id
}
```

In contrast to variables file which must consist of only variables the outputs file may have local variables.
It is allowed to define local variables in single `locals` block if and only if they are used only for outputs.
This special case is to reduce code duplication, ie. when module outputs same value, but in two different formats.
The local block must be at begin of the file, separated by single newline from first output block.
Every local value defined in outputs file must have `output_` prefix to denote their usage.

### Naming rules

Naming rules for outputs are similar to those for variables: clear, short, but meaningful.
Avoid prefixes, unless necessary.

## Resources & data blocks and local variables

Resource, data blocks and local variables should be defined in `main.tf` file.
If module is large, it is allowed to break this file into smaller one (see section below).

It is advised, but not enforced to put `locals` block first, followed by data blocks, then resources and module calls.
Blocks must be separated by single empty line or double empty line to group them.

```hcl
terraform {
  # explained above
}

locals {
  region = data.aws_region.current.name
}

data "aws_region" "current" {
  # no arguments
}

resource "aws_vpc" "main" {
  # arguments
}

module "nat_gw" {
  source = ".."

  # arguments
}
```

### Naming rules

Resources and data-blocks are internal, so their names are not as important as names of variables and outputs.
Nonetheless, they still must be meaningful and obey few simple rules.
Keep in mind that renaming resources is trivial in code, but causes issues for users.

Pleonasms should be avoided in resource naming.
That means name like `aws_instance.bastion_instance` is wrong, `aws_instance.bastion` is completely sufficient.

As best generic and default name, **main** should be used wherever possible, that means `aws_instance.main` is recommended.

When it comes to local variables - naming is most relaxed: just keep them understandable.

## Multiple `*.tf` files

When the `main.tf` file becomes too large to navigate easily, it is recommended to split it into multiple files.
It allows to locate resources faster and focusing on what you are working on better.
There are no explicit rules when you should do it, however there are rules where you **should no do it**:

* Creating file that is too small.
  File is considered small if it has less than 25 lines of code or less than 3 objects in it.
* Creating files based on their type, rather than composition.
  For example if module creates multiple S3 buckets with policies, it is better to put bucket+policy pair per file rather than all buckets in one and policies in another.
* Creating `data.tf` file with data resources only.
  In most cases it is bad idea, unless they are strongly coupled together.
  Even then, it's probably better to call it somehow different.
* Creating `locals.tf` with local variables only.
  The same reason as as above.
* Creating `versions.tf` file. Required versions specification should be in `main.tf` or in `terraform.tf` (for root-modules only).
* Creating `providers.tf`.
  The same reason as as above.

### Naming

The name of additional file should use "kebab-case", that is only lower case, each space replaced by a dash (`-`) character.
The name must roughly describe what's inside, add header-comment if needed.

## Creating examples

Each module must have at least one example of usage.
Each example must be in subdirectory of `examples/` with easily to understood name.
The codebase must meet the requirements for "root modules" described above.
The example directory must be fully working code, with all required variables be provided.
After downloading repository it should be possible to run `terraform plan` for every example provided.

Examples must not "over-complicate" provider configuration.
Whenever it is possible, example should rely on environmental variables rather than introduce variables.

Examples must not define state backend.

Examples must use simple and short name, and proper source of module defined in the first line.
```hcl
module "example" {
  source = "github.com/opsd-io/terraform-module-example"

  # arguments
}
```
