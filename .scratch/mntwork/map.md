# Chart the mntwork sandbox roadmap

Label: wayfinder:map

## Notes

Destination: A cross-platform CLI (`mntwork`) that provisions isolated Nix and container worktree sandboxes across Podman, Docker, and Apple's `container` CLI.

- Treat a named [Workspace](../../CONTEXT.md) as the unit of work. It may contain Worktree Contexts from multiple repositories.
- Protect the host outside explicitly mounted worktrees, their linked common Git directories, disposable caches, and explicit credential grants.
- Target macOS on Apple Silicon first. Linux support remains a design constraint for Podman and Docker, but not an initial validation claim.
- Keep the interactive shell separate from Sandbox Environment lifetime. A shell may detach while its environment continues running.
- Default generated Workspace data and disposable storage under the user's cache directory, while allowing the data root to move to an explicit path such as an external drive.
- Use `grill-with-docs` and `domain-modeling` when resolving every design ticket. Record accepted vocabulary in `CONTEXT.md` and create ADRs only for hard-to-reverse trade-offs.
- Treat the supplied OMP shell commands as untested exploration inputs, not reference implementations.

## Decisions so far

<!-- Empty until a child ticket is resolved. -->

## Fog

- Implementation language and distribution mechanism cannot be chosen until the CLI and driver boundaries are clearer.
- Direct-launch conveniences for specific coding-agent harnesses may emerge after the interactive shell lifecycle is defined.
- Resource budgets, idle shutdown, and garbage-collection policy depend on observed runtime behavior.
- The first integrated prototype cannot be scoped until runtime, configuration, routing, Nix, and worktree contracts settle.
- Linux validation depth and release sequencing remain unclear until the macOS runtime proof is complete.
