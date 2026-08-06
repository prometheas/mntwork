# CLI lifecycle ergonomics research

Research date: 2026-08-05. Sources are current official documentation.

## Comparable tools

| Tool | Initialization and canonical definition | Discovery and missing definition | Runtime lifecycle |
| --- | --- | --- | --- |
| Devbox | `devbox init` explicitly creates committed `devbox.json` in current directory. | `devbox shell` searches current directory and parents, or accepts `--config`. Its documented workflow starts with `init`; shell is consumer, not implicit initializer. | `shell` enters environment; scripts/services are separate concepts. |
| devenv | `devenv init` explicitly creates `devenv.nix`, `devenv.yaml`, and `.gitignore`. Canonical definition lives with project. | Commands operate from initialized project. Missing configuration is not described as auto-created by `shell`. | `shell` enters; `up` starts processes; `processes down` stops them. |
| Docker Compose | User authors `compose.yaml`; no required `init`. | Without `-f`, Compose searches current directory and parents. `-f` permits explicit location. Missing file cannot produce an application. | `up` creates and starts as needed. Other commands manage or enter existing project resources. Project name provides stable runtime identity. |
| Vagrant | `vagrant init` explicitly creates `Vagrantfile` in current directory. File is intended for version control. | Every command searches current directory upward for `Vagrantfile`; `VAGRANT_CWD` overrides lookup start. | `up` creates/configures on first use and resumes later; `ssh` enters; `halt` stops; `destroy` deletes. Creation is explicit at configuration layer but convergent at machine layer. |
| Dev Containers | User or UI explicitly adds `.devcontainer/devcontainer.json` or root `.devcontainer.json`; repository copy is canonical. | Folder supplies discovery context. After configuration exists, `Reopen in Container` creates or attaches. Missing config triggers an explicit “Add Configuration” flow, not silent file generation. | Reopen combines start/attach. Separate “Attach to Running Container” flow handles externally managed containers. |
| Docker Sandboxes (`sbx`) | No workspace manifest required. `sbx run` can create from current directory; `sbx create` offers explicit background creation and requires workspace path. `--name` gives stable identity. | Workspace path defaults to current directory for `run`. Repeating same path reconnects; named sandboxes can reconnect from any directory. | `run` creates/reconnects and attaches; `create` creates without attaching; `exec` opens shell; `stop` pauses; `rm` destroys. VM contents persist until `rm`. |
| Podman machine | `podman machine init [name]` explicitly creates named VM and defaults name when omitted. No project-local manifest. | Lookup is by machine name, not current project path. | `init`, `start`, `ssh`, `stop`, and `rm` remain distinct. |

## Patterns worth copying

1. **Explicit definition creation.** Devbox, devenv, Vagrant, and Dev Containers never make project files as a side effect of entering or starting. This protects user intent.
2. **Upward discovery after initialization.** Devbox, Compose, and Vagrant let commands run anywhere beneath project/workspace root.
3. **Convergent start.** Vagrant `up`, Compose `up`, and `sbx run` create/start or resume using established identity. Users need not know current runtime state first.
4. **Strict attach remains useful.** Vagrant `ssh`, Docker Sandbox `exec`, and Dev Containers attach flows fail conceptually when no target exists; they do not redefine workspace membership.
5. **Definition and runtime state differ.** Project definition belongs at discoverable project root. Mutable VM/container state belongs to runtime-managed storage and survives stop until explicit destruction.

## Recommendation for `mntwork`

Introduce explicit `init`; do not let plain `start` create files or directories.

```sh
mkdir -p ~/workspaces/checkout-redesign
cd ~/workspaces/checkout-redesign
mntwork init --member ~/work/api --member ~/work/ui

mntwork start
mntwork enter
mntwork stop
mntwork destroy
```

Semantics:

- Current directory is user-chosen **Workspace Root**, an anchor/control directory; it need not contain constituent repos.
- `mntwork init` validates members, derives or prompts for slug (default: Workspace Root basename), then writes generated `.mntwork/workspace.toml` there. It must preview target path and refuse overwrite unless explicit flag is supplied.
- Workspace record is mechanically generated but visible and inspectable. No manual TOML authoring required.
- `mntwork` commands search current directory and parents for `.mntwork/workspace.toml`; `--workspace PATH` provides explicit selection from elsewhere.
- Runtime IDs, leases, routes, ports, and health remain mutable state under XDG state directory, keyed by stable Workspace ID. Cache/storage remain under configurable data root. Do not mix these with Workspace Root record.
- `start` is convergent after initialization: create missing Sandbox Environment, restart stopped one, or attach a new interactive shell to running one.
- If record is missing, `start` exits without mutation and prints exact `mntwork init ...` guidance. Optional future `start --init` may combine steps, but mutation remains explicit in command line.
- `enter` is strict: only attach to running environment. `stop` preserves state. `destroy` removes runtime state; whether it removes local generated record should require explicit policy/flag because record is user-visible filesystem content.

This model blends Vagrant/Devbox discoverability with Docker Sandbox persistence. Workspace location is never guessed, `start` stays friendly, and generated composition remains distinct from disposable runtime state.

## Sources

- [Devbox quickstart](https://www.jetify.com/docs/devbox/quickstart)
- [Devbox shell reference](https://www.jetify.com/docs/devbox/cli-reference/devbox-shell/index)
- [devenv getting started](https://devenv.sh/getting-started/)
- [Docker Compose CLI reference](https://docs.docker.com/reference/cli/docker/compose/)
- [Docker Compose application model](https://docs.docker.com/compose/intro/compose-application-model/)
- [Vagrant init](https://developer.hashicorp.com/vagrant/docs/cli/init)
- [Vagrantfile lookup](https://developer.hashicorp.com/vagrant/docs/vagrantfile)
- [Vagrant up](https://developer.hashicorp.com/vagrant/docs/cli/up)
- [Vagrant ssh](https://developer.hashicorp.com/vagrant/docs/cli/ssh)
- [Create a Dev Container](https://code.visualstudio.com/docs/devcontainers/create-dev-container)
- [Docker Sandboxes usage](https://docs.docker.com/ai/sandboxes/usage/)
- [Docker Sandboxes architecture](https://docs.docker.com/ai/sandboxes/architecture/)
- [Podman machine init](https://docs.podman.io/en/stable/markdown/podman-machine-init.1.html)
