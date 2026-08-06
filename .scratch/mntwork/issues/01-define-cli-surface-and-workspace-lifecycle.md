# Define the CLI surface and Workspace lifecycle

Type: grilling
Status: resolved

## Question

What command surface and state machine let a user create, enter, detach from, inspect, stop, and destroy a named Workspace containing one or more Worktree Contexts, while supporting `--runtime` selection, an interactive shell by default, and optional direct launch into an agent harness?

## Comments

- Research: [CLI lifecycle ergonomics](../research/cli-lifecycle-ergonomics.md)
- Research: [Destroying running resources](../research/destroy-running-resource-ergonomics.md)
- Decision: [Initialize Workspace definitions explicitly](../../../docs/adr/0001-explicit-workspace-initialization.md)
- Decision: [Keep Workspace slugs unique per user](../../../docs/adr/0002-user-unique-workspace-slugs.md)
- Decision: [Use resource-first CLI grammar](../../../docs/adr/0003-resource-first-cli.md)
- Decision: [Layer runtime selection](../../../docs/adr/0004-layer-runtime-selection.md)
- Decision: [Scope Workspace deletion explicitly](../../../docs/adr/0005-scope-workspace-deletion.md)
- Decision: Sandbox State has four stable values: absent, stopped, running, and stale.
- Decision: [Never force-stop a Sandbox silently](../../../docs/adr/0006-never-force-stop-silently.md)
- Decision: [Require explicit destruction of a running Sandbox](../../../docs/adr/0007-require-explicit-running-sandbox-destruction.md)
- Decision: MVP launches agent harnesses through generic `sandbox exec`; named harness profiles remain future integration work.
- Decision: Read commands and mutation dry runs provide versioned JSON; progress uses stderr and exit codes remain stable for automation.

## Answer

mntwork uses resource-first grammar with `workspace` (`ws`) and `sandbox` (`sb`) resources. Global `--workspace` (`-w`) selects a registered user-global slug or Workspace Root path; without it, commands search the current directory and its parents for `.mntwork/workspace.toml`. Global `--runtime` (`-r`) overrides the Workspace runtime preference.

`workspace init [ROOT] --member PATH...` is the only initializer. An omitted Root means the existing current directory; an explicit missing Root is previewed and created. Initialization writes `$WORKSPACE_ROOT/.mntwork/workspace.toml`, registers a stable opaque Workspace ID and user-unique Workspace Slug, and never occurs implicitly from a Sandbox command. The Root is mounted read-write and becomes the default in-sandbox working directory; external members receive separate mounts.

Workspace Definition, Workspace Index, Workspace State, and heavy data remain distinct. Runtime selection resolves from CLI override, Workspace preference, user-global auto policy, then built-in capability detection. The resolved driver lives in Workspace State and changing it requires explicit Sandbox recreation.

The MVP Workspace commands are `init`, `list`, `show`, `member-add`, `member-list`, `member-remove`, `remove`, and `forget`. Membership changes refuse while a Sandbox is running or transitioning. `member-remove` safely deletes the linked Git worktree by default; `--keep-worktree` removes membership only. `workspace remove` deletes mntwork-owned definition/index/state by default and accepts `--delete worktrees|root|all`; it never deletes common Git repositories. `forget` removes only stale user-index registration.

The MVP Sandbox commands are `start`, `shell`, `exec`, `status`, `stop`, and `destroy`. `start` converges to running without attaching; `shell` converges and attaches interactively; `exec` requires running unless `--start` explicitly permits convergence. Stable Sandbox States are absent, stopped, running, and stale.

`sandbox stop` is graceful and never escalates silently; `--force` explicitly permits hard termination. Plain `sandbox destroy` refuses running, transitional, or attached Sandboxes; `--stop` opts into graceful stop then destroy, while `--force` permits hard stop/disconnection then destroy. `--yes` skips confirmation, and `--dry-run` resolves and prints exact targets without mutation.

Read commands and mutation dry runs provide versioned JSON. Human output is default, progress uses stderr, and exit codes remain stable for automation. MVP launches any agent harness through generic `sandbox exec`; named harness profiles remain later credential/integration work.
