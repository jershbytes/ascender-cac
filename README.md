# Ascender Configuration as Code

Ansible playbook for managing Ascender / Automation Controller configuration from YAML. The playbook applies the definitions in `vars/` to a controller, creating or updating objects so the configuration can be maintained in version control.

## Managed configuration

The playbook includes example definitions for:

- Organizations, teams, and users
- Machine and Vault credentials
- SCM projects and execution environments
- Inventories and SCM inventory sources
- Job templates and schedules

Definitions are stored in the matching files under `vars/`. All YAML files in that directory are loaded by `playbooks/deploy-platform.yml`. Keep referenced organization, project, inventory, credential, and execution environment names consistent across those files.

## Requirements

- Python and `ansible-core`
- Network access to the Ascender / Automation Controller API
- A controller account allowed to manage the configured objects
- The `ansible.controller` collection, version 4.0.0 or newer

Install the collection with:

```sh
ansible-galaxy collection install -r requirements.yml
```

## Configure access

The playbook connects to the controller using these environment variables:

| Variable | Purpose |
| --- | --- |
| `CONTROLLER_HOST` | Controller URL, such as `https://controller.example.com` |
| `CONTROLLER_USERNAME` | Controller account username |
| `CONTROLLER_PASSWORD` | Controller account password or token accepted by the API |
| `MACHINE_SSH_PRIVATE_KEY` | Private key used by the configured Machine credential |
| `ANSIBLE_VAULT_PASSWORD` | Password used by the configured Vault credential |

Set them in your shell or a secrets manager before running the playbook. Do not commit credentials or private keys. The playbook validates TLS certificates, so the controller's certificate must be trusted by the machine running Ansible.

## Run

From the repository root, export the required values and run:

```sh
export CONTROLLER_HOST="https://controller.example.com"
export CONTROLLER_USERNAME="your-automation-account"
export CONTROLLER_PASSWORD="..."
export MACHINE_SSH_PRIVATE_KEY="..."
export ANSIBLE_VAULT_PASSWORD="..."

ansible-playbook playbooks/deploy-platform.yml -i inventory.yaml
```

The playbook targets `localhost` and uses a local connection. Running without tags applies every configured object. To apply one area, use one of these tags:

| Tag | Configuration |
| --- | --- |
| `orgs` | Organizations |
| `teams` | Teams |
| `users` | Users |
| `credentials` | Credentials |
| `projects` | SCM projects |
| `execution_environments` | Execution environments |
| `inventories` | Inventories |
| `inventory_sources` | Inventory sources |
| `job_templates` | Job templates |
| `schedules` | Schedules |

For example, to apply only project definitions:

```sh
ansible-playbook playbooks/deploy-platform.yml -i inventory.yaml --tags projects
```

## Customize

The values in `vars/` are examples, not ready-to-deploy settings. Before the first run, update the organization, project URLs, playbook and inventory paths, credentials, execution environment, and schedule to match your controller and repositories. In particular, replace the `your-org` GitHub URLs in `vars/projects.yml`.

The GitHub Actions workflow is manual-only by default. To use it, add repository secrets named `CONTROLLER_HOST`, `CONTROLLER_USERNAME`, `CONTROLLER_PASSWORD`, `MACHINE_SSH_PRIVATE_KEY`, and `ANSIBLE_VAULT_PASSWORD`, then run **Deploy Controller Configuration** from the Actions tab. Review the example definitions before running it.

The schedule example is disabled by default. If enabled, it runs weekly on Tuesday at 3:00 AM UTC, starting November 3, 2026; edit it in `vars/schedules.yml` to suit your needs.

The collection source configuration is in `ansible.cfg`, and the collection requirement is in `requirements.yml`.