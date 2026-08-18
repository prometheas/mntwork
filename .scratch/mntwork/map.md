# Chart the mntwork sandbox roadmap

Label: wayfinder:map

## Notes

Destination: A cross-platform CLI (`mntwork`) that provisions isolated Nix and container worktree sandboxes across Podman, Docker, and Apple's `container` CLI.

- Treat a named [Workspace](../../CONTEXT.md) as the unit of work. It may contain Worktree Contexts from multiple repositories.
- Protect the host outside explicitly mounted worktrees, their linked common Git directories, disposable caches, and explicit credential grants.
- Target macOS on Apple Silicon first. Linux support remains a design constraint for Podman and Docker, but not an initial validation claim.
- Keep the interactive shell separate from Sandbox Environment lifetime. A shell may detach while its environment continues running.
- Default generated Workspace data and disposable storage under the user's cache directory, while allowing the data root to move to an explicit path such as an external drive.
- Support host 1Password as a credential broker for in-sandbox Git/SSH signing and narrowly granted secrets without copying private key material into the Sandbox Environment.
- Use `grill-with-docs` and `domain-modeling` when resolving every design ticket. Record accepted vocabulary in `CONTEXT.md` and create ADRs only for hard-to-reverse trade-offs.
- Treat the supplied OMP shell commands as untested exploration inputs, not reference implementations.

## Decisions so far

- [Define the CLI surface and Workspace lifecycle](issues/01-define-cli-surface-and-workspace-lifecycle.md) — Explicit Workspace initialization and resource-first `workspace`/`sandbox` commands separate durable composition from safe, scriptable Sandbox lifecycle.
- [Define the Runtime Driver interface](issues/02-define-runtime-driver-interface.md) — Capability-gated preparation feeds strict lifecycle primitives, exact mount grants, sandbox-scoped workload engines, explicit host channels, and verifiable reconciliation and cleanup.
- [Design the Nix image and artifact reuse model](issues/04-design-nix-image-and-artifact-reuse.md) — Digest-pinned base images, persistent private Workspace stores, exact environment authorization, and an optional read-only signed cache separate trust from reuse.
- [Design the Worktree Inspector](issues/05-design-worktree-inspector.md) — Explicit Git-plumbing inspection produces host-scoped location identities, continuity and live-equivalence evidence, atomic repository leases, and an exact Logical Mount Grant Set without repository scanning or mutation.
- [Define Repository Manifest and Workspace Definition contracts](issues/06-define-repository-manifest-and-workspace-definition.md) — Manifests state project needs only, merge by declared per-field semantics under a user-global Manifest Request Ceiling, and drift re-authorizes only on increase; Definition holds portable intent while the Index holds every host path.

## Fog

- Resource budgets, idle shutdown, and garbage-collection policy depend on observed runtime behavior.
- Concrete ceiling defaults and the initial manifest-requestable capability allowlist depend on the macOS runtime proof.
- The first integrated prototype cannot be scoped until routing and the macOS runtime proof settle; the CLI, driver, Nix, worktree, and configuration contracts are now fixed.
- Linux validation depth and release sequencing remain unclear until the macOS runtime proof is complete.
