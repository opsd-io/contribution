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
Most important aspects are described below.

* `.envrc` - [direnv](https://direnv.net/) configuration.
* `.pre-commit-config.yaml` - [pre-commit]() configuration.
* `.terraform-docs.yml` - [terraform-docs]() configuration.
* `.tflint.hcl` - [tflint](https://github.com/terraform-linters/tflint) configuration.
* `.tool-versions` - [asdf](https://asdf-vm.com/) configuration.
* `LICENSE` - license under which this module is available.
* `README.md` - up to date description of the module, what it should be used for and how. This file should is updated by *terraform-docs* tool.
* `variables.tf` - all module variables must be in this file. If there are none, file should be removed.
* `main.tf` - this file should contain all data sources, resources, module calls and local variables.
  If this file becomes too big to work with or resources could be grouped together it should be spitted to multiple `*.tf` files - see rules below.
* `outputs.tf` - all module outputs must be in this file. If there are none, file should be removed.
* `modules/` - directory for nested modules. They must obey the same rules as main module.
* `examples/` - directory for examples of using this module. Each example should be placed in separate subdirectory.
  The example directory should have README file that explains what this example is about.

## Input variables

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

## Output values

```hcl
output "name" {
  description = "The ID of null_resource instance."
  sensitive   = false
  value       = null_resource.test.id
}
```

## Resources & data blocks
