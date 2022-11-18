# Terraform modules

## Repository structure

The below diagram shows a typical terraform module repo structure. To not reinvent the wheel, we adopted the [schema](https://www.terraform.io/language/modules/develop/structure) proposed by Terraform team.

```
.
├── README.md
├── main.tf
├── variables.tf
├── outputs.tf
├── ...
├── modules/
│   ├── nestedA/
│   │   ├── README.md
│   │   ├── variables.tf
│   │   ├── main.tf
│   │   ├── outputs.tf
│   ├── nestedB/
│   ├── .../
├── examples/
│   ├── exampleA/
│   │   ├── main.tf
│   ├── exampleB/
│   ├── .../
```

## Tools

For local development, you need a few things:

- [asdf](https://asdf-vm.com/) - which manages multiple runtime versions;
- [direnv](https://direnv.net/) - which augments existing shells with a new feature that can load and unload environment variables.

All these tools must be installed and configured according to their official documentation.

### asdf

The config file `.tool-versions` is located in the root directory. The structure is very straightforward.

```shell
terraform 1.2.9
terraform-docs 0.16.0
tflint 0.40.0
pre-commit 2.20.0
```

The full packages list can be found in the [.tool-versions](.tool-versions) file.

Before we install the tools, we need to add the plugins:

```shell
asdf plugin add terraform
asdf plugin add terraform-docs
asdf plugin add tflint
asdf plugin add pre-commit
```

To install required tools, run the command as in the example below

```shell
asdf install
```

**Important**: Please keep in mind the versions or tools themselves might change, so it is advised to rerun `asdf install` command after pulling from git.

After the installation process, all the tools will be installed. You can verify this by executing

```shell
% terraform --version
Terraform v1.2.9
on darwin_amd64
```

in the project directory.

More information and examples can be found in the ASDF project [documentation](https://asdf-vm.com/manage/plugins.html).

### direnv

The direnv config files `.envrc` might be present in many directories.
Therefore, you must approve each of them by entering a particular directory whenever their contents change.

**Notice**: The descriptive error will pop up if you forget to do so.

To allow direnv to load its config run:

```shell
% direnv allow
direnv: loading .envrc
direnv: export ~PATH
```
**Notice**: output will vary depending on directory in which you run it.

### Pre-commit hooks

The repository has `pre-commit` hooks configured:

```yaml
repos:
  # Pre-commit hooks
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.3.0 # Get the latest from https://github.com/pre-commit/pre-commit-hooks/releases
    hooks:
      - id: end-of-file-fixer
        name: end of file fixer
        description: Let's be sure that a new line has been added to the end of the file.
      - id: trailing-whitespace
        name: trailing whitespace
        description: Automatically remove trailing whitespace before committing.
      - id: check-merge-conflict
        name: check merge conflict
        description: Check for files that contain merge conflict strings.
        stages: [commit]
      - id: check-executables-have-shebangs
        name: check executables have shebangs
        description: Checks that non-binary executables have a proper shebang.
        stages: [commit]
      - id: detect-private-key
        name: detect private key
        description: Checks for the existence of private keys.
        stages: [commit]
      - id: check-symlinks
        name: check symlinks
        description: Checks for symlinks which do not point to anything.
        stages: [commit]
      - id: mixed-line-ending
        name: mixed line ending
        description: Replaces or checks mixed line ending.
        stages: [commit]
      - id: check-yaml
        name: check yaml
        description: checks yaml files for parseable syntax.
        entry: check-yaml
        language: python
        types: [yaml]

  # Terraform
  - repo: https://github.com/antonbabenko/pre-commit-terraform
    rev: v1.75.0 # Get the latest from: https://github.com/antonbabenko/pre-commit-terraform/releases
    hooks:
      - id: terraform_fmt
        name: terraform fmt
        description: Checks if the terraform code format is valid.
        stages: [commit]
      - id: terraform_tflint
        name: terraform tflint
        description: Automatic terraform linting.
        stages: [commit]
        exclude: (examples)
      - id: terraform_validate
        name: terraform validate
        description: Terraform code validator.
        stages: [commit]
        exclude: (examples)
      - id: terraform_docs
        name: terraform docs
        description: Generates terraform documentation.
        args:
          - --args=--config=.terraform-docs.yml
        stages: [commit]
```

The full hooks list can be found in the [.pre-commit-config.yaml](.pre-commit-config.yaml) file.

**Important**: We're using **GitHub Actions** to trigger `pre-commit` checks on the **Pull Request**. The configuration file can be found in the in the [static-code-analysis.yml](.github/workflows/static-code-analysis.yml) file.

## Versioning

The module should be tagged using SemVer and a changelog. Use the [semantic commits](https://www.conventionalcommits.org/en/v1.0.0/) so that we can auto-generate the changelog.