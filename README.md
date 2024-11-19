# Nautobot Bootstrap Data

## Description

This repo contains the data that will sync using the Bootstrap SSoT in Nautobot to create baseline environments. Most items will receive a custom field associated with them called "System of Record", which will be set to "Bootstrap". These items are then the only ones managed by the Bootstrap SSoT App. Other items within the Nautobot instance will not be affected unless there's items with overlapping names. There is currently one exception to this and that is the ComputedField model since it can't have a custom field associated with it due to the fact that it's a child of CustomField itself. If you choose to manage ComputedField objects with the Bootstrap SSoT App, make sure to define them all within the yaml file, since any "locally defined" computed fields within Nautobot will end up getting deleted when the job runs.

## Configuration

### Bootstrap data

Bootstrap data can be stored in 2 fashions. Firstly, it can be stored within the `nautobot_ssot_bootstrap/fixtures` directory within the Bootstrap SSoT plugin itself, or you may create a Git Repository within an existing Nautobot instance that contains the word `Bootstrap` in the name and provides `config context` data. The data structure is flat files, there is a naming scheme to these files. The first one required is `global_settings.yml`. This contains the main data structures of what data can be loaded `Secrets,SecretsGroups,GitRepository,DynamicGroup,Tag,etc`. You can then create additional `.yml` files with naming of your CI environments, i.e. production, development, etc. This is where the environment variables described below would be matched to pull in additional data from the other yaml files defined in the directory. A simple structure would look something like this:

```text
global_settings.yml
develop.yml
prod.yml
staging.yml
```

There are 2 environment variables that control how certain things are loaded in the app.

  1. `NAUTOBOT_BOOTSTRAP_SSOT_LOAD_SOURCE` - defines whether to load from the local `fixtures` folder or a GitRepository already present in Nautobot.
    - Acceptable options are `file` or `git`.
  2. `NAUTOBOT_BOOTSTRAP_SSOT_ENVIRONMENT_BRANCH` - Defines the environment and settings you want to import. I.e. production, develop, staging.

## Usage

### Pre-commit hook

You must run the pre-commit hook before commiting new updates to the repository to ensure yaml is valid. This does not check the accuracy of the data, just that it is valid yaml. If you want to check your work before committing, make sure you are in the `poetry shell`, you have installed the prerequisites with `poetry install`, then run `pre-commit install` to setup the pre-commit environment. Lastly  run `pre-commit run --all-files` to verify yaml is valid. This will also run on commit and prevent the commit from completing if there are any errors.

### Data structures

#### global_settings.yml

```yaml
secret:
  - name: Github_Service_Acct
    provider: environment-variable # or text-file
    parameters:
      variable: GITHUB_SERVICE_ACCT
      path:
  - name: Github_Service_Token
    provider: environment-variable # or text-file
    parameters:
      variable: GITHUB_SERVICE_TOKEN
      path:
secrets_group:
  - name: Github_Service_Account
    secrets:
      - name: Github_Service_Acct
        secret_type: username
        access_type: HTTP(S)
      - name: Github_Service_Token
        secret_type: token
        access_type: HTTP(S)
git_repository:
  - name: Backbone Config Contexts
    url: https://github.com/nautobot/backbone-config-contexts.git
    secrets_group_name: Github_Service_Account
    provided_data_type:
      - config contexts
dynamic_group:
  - name: Backbone Domain
    content_type: dcim.device
    filter: |
      {
        "tenant": [
          "backbone"
        ]
      }
computed_field:
  - label: Compliance Change
    content_type: nautobot_golden_config.configcompliance
    template: '{{ obj | get_change_log }}'
tag:
  - name: Backbone
    color: '795548'
    content_types:
      - dcim.device
```

#### develop.yml

```yaml
git_branch: develop
```
