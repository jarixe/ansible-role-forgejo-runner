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
| `forgejo_runner_container_docker_host` | `""` | act_runner's `container.docker_host`. `""` withholds the host Docker socket from job containers; `"automount"` mounts it in, which a workflow that runs `docker build`/`docker push` requires. See the caveat below. |
| `forgejo_runner_container_valid_volumes` | `[]` | Host paths a job container may mount. Pair with `automount` as `["/var/run/docker.sock"]`. |
| `forgejo_runner_instance_url` | `http://localhost:3000` | URL of the Forgejo instance this runner connects to. |
| `forgejo_runner_uuid` | `""` | Runner UUID from Forgejo (**required**, shown only once when the runner is created). |
| `forgejo_runner_token` | `""` | Runner token from Forgejo (**required**, shown only once when the runner is created). |

The play fails fast with an assertion if `forgejo_runner_uuid` or `forgejo_runner_token` is left empty.

### Docker-in-job caveat

By default a job container cannot reach the host's Docker daemon, so a workflow step that shells out to `docker` fails with `dial unix /var/run/docker.sock: connect: no such file or directory`. Setting `forgejo_runner_container_docker_host: "automount"` fixes that, but it hands **every** containerised job on this runner root-equivalent control of the host daemon — a job can mount any host path or start a privileged container. Enable it only on a runner whose repositories you trust to the same degree as the host, and prefer scoping image builds to a dedicated runner where that is not true.

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
