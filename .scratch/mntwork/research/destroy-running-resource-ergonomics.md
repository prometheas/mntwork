# Destroying running resources: CLI ergonomics research

## Comparable tools

| Tool | Running target without an override | Override behavior | Confirmation / dry run |
| --- | --- | --- | --- |
| Vagrant `destroy` | Stops a running machine and destroys its resources. | `--graceful` explicitly requests a graceful shutdown; `--force` means skip confirmation, not "hard kill." | Prompts by default; `--force` skips the prompt. No dry-run option is documented. |
| Docker `container rm` | Refuses removal of a running container. | `--force` sends `SIGKILL`, then removes it. | No confirmation or dry-run option is documented. |
| Docker Compose `down` | Stops running services, then removes the Compose containers and networks. | `--timeout` controls the shutdown wait; there is no `--force` option on `down`. | No confirmation prompt is documented. Compose has a global `--dry-run` option. |
| Podman `rm` | Refuses running or paused containers. | `--force` permits removal; `--time` optionally waits before forcibly stopping and is valid only with `--force`. | No confirmation or dry-run option is documented. |
| Podman `machine rm` | Ordinary removal previews the files and asks for confirmation; the docs reserve removal of a running machine for `--force`. | `--force` stops and deletes the VM without confirmation. | Prompts by default; no dry-run option is documented. |
| Docker Sandboxes `sbx rm` | Stops a running sandbox and removes all associated resources, but refuses if the sandbox has an active attach, SSH, or SFTP session. | `--force` skips confirmation and permits deletion while in use. | Prompts by default; no dry-run option is documented. |
| Multipass `delete` | The official tutorial deletes a running VM directly. Deletion first moves the instance into recoverable `Deleted` state. | `--purge` makes deletion immediately permanent; otherwise a later `purge` permanently removes deleted instances. | No confirmation or dry-run option is documented. The recoverable intermediate state is its safety mechanism. |

## What the analogues establish

There is no universal convention:

- Low-level container deletion (`docker rm`, `podman rm`) treats "running" as a state error unless the operator explicitly requests force.
- Environment-level teardown (`vagrant destroy`, `docker compose down`, `sbx rm`) commonly owns both stop and delete, so it may stop a running target automatically.
- Active human or agent use is a distinct safety boundary: Docker Sandboxes refuses normal deletion when a session is attached even though it otherwise stops running sandboxes automatically.
- `--force` is inconsistent across tools. It can mean hard kill, permission to delete an in-use target, or merely "do not prompt." `mntwork` should define it narrowly rather than inherit that ambiguity.
- Multipass demonstrates a useful alternative: deletion can be recoverable until a separate purge, though that does not map cleanly to all runtime backends.

## Recommendation for `mntwork`

Because an `mntwork` Sandbox Environment may host autonomous work and multiple attached shells, default `destroy` should favor state safety over the convenience convention used by Vagrant and Compose:

```sh
# Stopped Sandbox Environment: preview, confirm, then destroy.
mntwork -w checkout-redesign sb destroy

# Running Sandbox Environment: explicitly opt into graceful stop then destroy.
mntwork -w checkout-redesign sb destroy --stop

# Hard-stop a refusing or in-use Sandbox Environment, then destroy.
mntwork -w checkout-redesign sb destroy --force

# Resolve targets and show the complete plan without mutation.
mntwork -w checkout-redesign sb destroy --dry-run
```

Contract:

1. Plain `sb destroy` refuses a running, starting, stopping, or actively attached Sandbox Environment and prints the exact safe next commands.
2. `--stop` requests graceful shutdown, waits the configured timeout, and destroys only after shutdown succeeds. Timeout aborts without deletion.
3. `--force` alone means permission to hard-stop and disconnect active sessions before destruction. It must not double as "skip confirmation"; use the existing `--yes` automation control for that.
4. `--dry-run` performs runtime reconciliation and prints every environment, route, lease, volume, and workspace-scoped storage target that would be removed.
5. Workspace Definition, Workspace Index registration, Workspace Root, and member worktrees remain outside `sb destroy` scope.

This gives users the explicit two-step model they expect while retaining a deliberate Compose-like convenience path through `--stop`.

## Sources

- [Vagrant `destroy`](https://developer.hashicorp.com/vagrant/docs/cli/destroy)
- [Docker `container rm`](https://docs.docker.com/reference/cli/docker/container/rm/)
- [Docker Compose `down`](https://docs.docker.com/reference/cli/docker/compose/down/)
- [Docker Compose global options and dry run](https://docs.docker.com/reference/cli/docker/compose/)
- [Docker Compose stop grace period](https://docs.docker.com/reference/compose-file/services/#stop_grace_period)
- [Podman `rm`](https://docs.podman.io/en/latest/markdown/podman-rm.1.html)
- [Podman `machine rm`](https://docs.podman.io/en/latest/markdown/podman-machine-rm.1.html)
- [Docker Sandboxes `sbx rm`](https://docs.docker.com/reference/cli/sbx/rm/)
- [Multipass tutorial](https://documentation.ubuntu.com/multipass/latest/tutorial/)
- [Multipass `delete`](https://documentation.ubuntu.com/multipass/latest/reference/command-line-interface/delete/)
