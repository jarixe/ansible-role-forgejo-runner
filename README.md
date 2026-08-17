# Ansible Role: Forgejo Runner

Installs and configures a [Forgejo Actions](https://forgejo.org/docs/latest/admin/actions/) runner as a native systemd service (not containerised).

Authenticates via Forgejo v12's `server.connections` (UUID + token), so there is no `register`/`.runner` step — the runner must be pre-created in Forgejo first (Site Admin → Actions → Runners → Create new runner), which shows the UUID and token once.

## Requirements

- Docker must already be installed on the target host — the runner user is added to the `docker` group, and jobs run via `docker://` label images over the host network.
- No external collections required.

## Role Variables

| Variable | Default | Description |
|---|---|---|
| `forgejo_runner_version` | `12.12.0` | Forgejo runner release version to install. |
| `forgejo_runner_download_base` | `https://code.forgejo.org/forgejo/runner/releases/download` | Base URL for release downloads. |
| `forgejo_runner_url` | derived | Full download URL for the linux-amd64 binary. |
| `forgejo_runner_user` | `forgejo-runner` | System user the service runs as. |
| `forgejo_runner_group` | `forgejo-runner` | System group for the service user. |
| `forgejo_runner_install_dir` | `/opt/forgejo-runner` | Install directory (binary, config, cache, workspace). |
| `forgejo_runner_capacity` | `2` | Number of concurrent jobs the runner accepts. |
| `forgejo_runner_labels` | `["docker:docker://node:22-bookworm", "ubuntu-latest:docker://catthehacker/ubuntu:act-latest"]` | Labels advertised by the runner, mapped to Docker images. |
| `forgejo_runner_instance_url` | `http://localhost:3000` | URL of the Forgejo instance this runner connects to. |
| `forgejo_runner_uuid` | `""` | Runner UUID from Forgejo (**required**, shown only once when the runner is created). |
| `forgejo_runner_token` | `""` | Runner token from Forgejo (**required**, shown only once when the runner is created). |

The play fails fast with an assertion if `forgejo_runner_uuid` or `forgejo_runner_token` is left empty.

## Example Playbook

```yaml
- hosts: ci_runners
  become: true
  roles:
    - role: ansible-role-forgejo-runner
      vars:
        forgejo_runner_instance_url: "https://forgejo.example.com"
        forgejo_runner_uuid: "{{ vault_forgejo_runner_uuid }}"
        forgejo_runner_token: "{{ vault_forgejo_runner_token }}"
```

## License

MIT
