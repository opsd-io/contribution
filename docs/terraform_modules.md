# Terraform general

## Prerequisites

Read the [Terraform general](terraform_general.md) contributing guide, and set up your development environment.

## Creating a repository

Use [terraform-module-template](https://github.com/opsd-io/terraform-module-template) to create new Terraform module.
The name of the repository should match one of the formats:

* `terraform-module-$provider-$resource` - for basic modules that wrap provider resources with or without additional resources.
  Additional resources must be part of the main resource, owned exclusively and strongly affiliated. Good examples of composite modules are virtual machines with additional disks, virtual networks with sub-networks, or database clusters with multiple nodes.

* `terraform-module-$provider-$functionality` - for more complex aggregation of resources that solves particular use cases.
  This type of module comes with a limited configuration of resources to fit specific situations, even if the resource is capable of more.
  These modules may, but do not have to, use generic modules, and the presence of one does not exclude the existence of others.
  An example would be *terraform-module-aws-static-website* that combines CloudFront CDN and S3 buckets to serve static files, or *terraform-module-aws-backend* that combines S3 bucket with DynamoDB to provide Terraform state backend storage.
  Even if both mentioned modules use S3, they do it in different ways, requiring different configuration options. Still, generic *terraform-module-aws-s3-bucket* might exist, with a most complex set of configuration options.

Please ensure the repository settings are set: pull request options and branch protection rules.
<!-- TODO: write more about it. -->

## Repository structure

The OPSd module repository structure is based on [Standard Module Structure](https://developer.hashicorp.com/terraform/language/modules/develop/structure).
The most important aspects are described below:
* `LICENSE` - license under which this module is available.
* `README.md` - up to date description of the module, what it should be used for and how. This file should be updated by the *terraform-docs* tool.
* `variables.tf` - all module variables must be in this file. If there are none, the file should be removed.
* `main.tf` - this file should contain all data sources, resources, module calls and local variables.
  If this file becomes too big to work with or resources could be grouped, it should be split into multiple `*.tf` files - see rules below.
* `outputs.tf` - all module outputs must be in this file. If there are none, the file should be removed.
* `modules/` - directory for nested modules. They must obey the same rules as the main module.
* `examples/` - directory for examples of using this module. Each example should be placed in a separate subdirectory.
  The example directory should have a README file that explains what this example is about.

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
A newline at the end of the file is mandatory.

Remember to `terraform fmt` - before committing your code.
Check code syntax and conventions with `flint`.
Both tools are automatically called by `pre-commit` hooks, and their status is also validated in pull request checks.
**Notice**: Some rules listed here are not enforced by syntax checking tools; you have to verify this on your own manually.

## Code formatting rules

1. There must be an empty line between any two blocks, both top-level and nested.
2. Arguments equal signs in blocks on the same level must be aligned and surrounded with at least space.
3. A nested block must be preceded by an empty line unless there are no arguments.
4. Additional empty lines are allowed to separate logical groups of arguments within a block.
5. The following order must be followed:
   1. special meta-arguments, no empty lines between them, but one line after them:
      1. the `source` and optionally `version` argument, only for `module` blocks
      2. the `provider` meta-argument, if the alternate provider is needed
      3. dynamic resource meta-arguments: like `count` or `for_each`
      4. the `depends_on` meta-argument, with each dependency in a new line
   2. arguments, optionally grouped by a single empty line
   3. nested blocks and dynamic blocks
   4. the `connection` and `provisioner` blocks (remember, only use service providers when there is no other option)
   5. special `lifecycle` meta-block.
6. No empty lines between closing brackets `}` at the end of nested blocks, maps or objects.

## Naming objects

Every variable, resource, module invocation and output must use snake case - only lowercase; each space is replaced by an underscore (`_`) character.
The names of variables and outputs as an interface of the module must be descriptive and straightforward to understand.
The names of resources might need to be clarified as an internal matter as long as they stay meaningful.
You can read more about variables, outputs and resources if you want more information.

## Comments

Comment should start with a single `#` sign, followed by a single space.
Usage of `//` is not allowed.
Multiline comments might be delimited by `/*` and `*/`, but it's preferable to use multiple `#` lines.

Avoid fancy formatting, ASCII-art boxes, multiple `#` signs at the beginning or any at the end of lines, etc.

It can add single or multiline comments at the beginning of a file to explain its purpose, but only if it adds additional information.
A single empty line must follow this comment.

It is also acceptable to add a single-line comment, surrounded by empty lines, that describes the resources below, creating something like a group.

The comment describing a particular resource should be right before it, without an empty line between the comment and the resource block.

Avoid unnecessary comments, which means comments that do not provide any additional information.
For example, the `aws_instance.bastion` comment should NOT be "The AWS instance for bastion host" - the resource type and name have already explained this.

In contrast to meaningless comments, some resources or expressions might require explanation.
This is true for complex firewall rules, policies, or resources whose purpose isn't clear in the context of the module.
The best example is multi-line expressions, especially for expressions (aka. list/map comprehension) that transform one structure to another.
We would highly recommend that you explain why and how it is made.

**Rule of thumb**: less is better. Add comments only if they might help someone else understand your code.

An example file with lots of comments:

```hcl
# This file is a pure abstract example.
# All resources here have no meaning; the code itself will not work!

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

# VPC routing tables & rules for the current environment.

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
The file starts with two lines of comments describing its purpose.
Both `aws_vpc.main` and `aws_s3_bucket.images` don't have a comment, as it's clear what they are for.
The name of `aws_s3_bucket.es_backups` seems obvious, but the comment gives additional context about how (snapshots) and where (RDS) this bucket is used for backup.
Next comment separates previous resources from routing tables and their rules. If there were more of them, they might be moved to a separate file, i.e. `routing.tf`, but this is not required.

Please notice the names of resources, too:
There is just one VPC managed here, so "main" is a good name.
Each S3 bucket resource has a short but meaningful name.
The route table names are simple as "public" and "private" to avoid code-smells like `aws_route_table.private_route_table` or blank names like "rt1" and "rt2".

## Terraform block

Terraform settings are gathered in the `terraform` block.
For modules themselves, this block should be at the beginning of the `main.tf` file.
Version constraints should define only their minimum allowed version unless it is known it will not work with a particular version (due to bugs) or is too new (because of known incompatibilities).

The first field in the block must be the `required_version` to define the Terraform runtime version. After the empty line, the `required_providers` block must be created with all providers required by this module defined in alphabetical order.
No empty lines are allowed inside the required provider block.

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
Providers that require no configuration (e.g. random or archive) or when all arguments are specified using environment variables should not have any blocks unless aliases are needed.

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

Despite the `backend` block being nested within the `terraform` block, its definition must be in a separate `backend.tf` file with all common settings like backend type and fixed configuration options.
The distinctions for each root module (ie. different path names) must be set in the `backend-config.tfvars` file.
The reasoning for this is that it is much easier to synchronize (copy-paste or symlink-friendly) this block among multiple root modules.

Module examples must not define a backend, silently using the default `local` backend, which stores state in current working directory.

## Input variables

All input variables must be in `variables.tf` file.
Each variable must have a valid name (see naming rules), description and type.
If input value is sensitive, `sensitive = true` is required, explicit false value is optional, but advised if any other inputs are sensitive.
If input value cannot be null, the `nullable = false` should be used, explicit true value is optional.

Using defaults is restricted to:
* if the variable is passed directly to the resource into the optional parameter, then `default = null` can be used.
* if variable type is `list(..)` or `set(..)`, and empty value is allowed, then `default = []` can be used.
* if variable type is `set(..)`, and empty value is allowed, then `default = {}` can be used.
* if the value can have any "good practice" or "safe" default value, it should be used.
  For example, the default port for service or the smallest reasonable OS disk size.

Input variable blocks must be separated by a single empty line or double empty line to group them, if needed.

**Order of arguments**: description, type, sensitive, nullable, default.
**Reason**: The description is close to the name; when default is last, multi-lined expressions are more readable.
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

Whenever the module manages a single resource, variable prefixes should be avoided.
For example, if the module manages a "foobar" resource, it's better to have `var.name` and `var.size` rather than `var.foobar_name` and `var.foobar_size` as long as it's straightforward.

When the module manages a few strongly associated resources, e.g., a virtual machine, an OS disk, and a network card, `var.name` is still good for the VM name because it is the primary resource that the module manages.
For os-disk and network card as supplementary resources, the `var.os_disk_name` and `var.nic_name` should be used.
This rule works when one resource is a "parent", and others are "children" that cannot exist independently.

When the module manages a few weakly associated resources, or it is hard to point to the "root" resource, it is advised to prefix both variables to avoid confusion.
For example, for a module that creates a compute cluster and load-balancer, the `var.name` is vague and should be avoided.
In this situation, the `var.cluster_name` and `var.lb_name` should be used.

## Output values

All module outputs must be in `outputs.tf` file.
Each output must have valid name (see naming rules) and description.
If output value is sensitive, `sensitive = true` is required, explicit false value is optional, but advised if any other outputs are sensitive.

No empty lines inside output block is allowed.
Output blocks must be separated by single empty line or double empty line to group outputs.

**Order of arguments**: description, sensitive, value.
**Reason**: The description is close to the name; when the value is last, multi-lined expressions are more readable.
The sensitive flag is in the middle as the only remaining option.

The complete output sample:
```hcl
output "id" {
  description = "The ID of null_resource instance."
  sensitive   = false
  value       = null_resource.test.id
}
```

In contrast to the variables file, which must consist of only variables, the outputs file may have local variables.
Local variables can be defined in a single `locals` block if and only if they are used only for outputs.
This special case reduces code duplication, i.e., when a module outputs the same value but in two different formats.
The local block must be at begin of the file, separated by a single newline from the first output block.
Every local value defined in outputs file must have `output_` prefix to denote their usage.

### Naming rules

Naming rules for outputs are similar to those for variables: clear, short, and meaningful.
Avoid prefixes, unless necessary.

## Resources & data blocks and local variables

Resource, data blocks and local variables should be defined in `main.tf` file.
If the module is large, it is allowed to break this file into smaller ones (see the section below).

It is advised, but not enforced to put `locals` block first, followed by data blocks, then resources and module calls.
Blocks must be separated by single or double empty lines to group them.

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

Resources and data blocks are internal, so their names are not as important as the names of variables and outputs.
Nonetheless, they still must be meaningful and obey a few simple rules.
Keep in mind that renaming resources is trivial in code but causes issues for users.

Pleonasms should be avoided in resource naming.
That means a name like `aws_instance.bastion_instance` is wrong; `aws_instance.bastion` is completely sufficient.

As the best generic and default name, **main** should be used wherever possible, which means `aws_instance.main` is recommended.

When it comes to local variables - naming is most relaxed: just keep them understandable.

## Multiple `*.tf` files

When the `main.tf` file becomes too large to navigate easily, it is recommended to split it into multiple files.
It allows you to locate resources faster and focus on what you are working on better.
There are no explicit rules when you should do it, however, there are rules where you **should not do it**:

* Creating a file that is too small.
  A file is considered small if it has less than 25 lines of code or less than 3 objects in it.
* Creating files based on their type, rather than composition.
  For example, if the module creates multiple S3 buckets with policies, it is better to put a bucket+policy pair per file rather than all buckets in one and policies in another.
* Creating `data.tf` file with data resources only.
  In most cases, it is a bad idea unless they are strongly coupled together.
  Even then, it's probably better to call it somehow different.
* Creating `locals.tf` with local variables only.
  The same reason as as above.
* Creating `versions.tf` file. Required version specification should be in `main.tf` or in `terraform.tf` (for root modules only).
* Creating `providers.tf`.
  The same reason as as above.

### Naming

The name of the additional file should use "kebab-case", which is only lowercase, each space replaced by a dash (`-`) character.
The name must roughly describe what's inside, and add header-comment if needed.

## Creating examples

Each module must have at least one example of usage.
Each example must be in a subdirectory of `examples/` with an easily understood name.
The codebase must meet the requirements for "root modules" described above.
The example directory must be fully working code, with all required variables be provided.
After downloading the repository, it should be possible to run the `terraform plan` for every example provided.

Examples must not "over-complicate" provider configuration.
Whenever it is possible, the example should rely on environmental variables rather than introduce variables.

Examples must not define the state backend.

Examples must use simple and short names and the proper source of the module defined in the first line.

```hcl
module "example" {
  source = "github.com/opsd-io/terraform-module-example"

  # arguments
}
```
