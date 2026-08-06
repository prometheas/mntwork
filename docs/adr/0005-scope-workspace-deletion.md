# Scope Workspace deletion explicitly

`workspace remove` deletes mntwork-owned definition, index, and state by default; `--delete worktrees|root|all` explicitly expands deletion to user filesystem resources, while common Git repositories are never targets. `--dry-run` exposes the complete plan, and `--force` bypasses content-safety checks only—it cannot expand targets, escape validated paths, or delete common repositories—so convenience teardown does not weaken target identity guarantees.

`workspace member-remove PATH` removes membership and safely deletes the linked Git worktree by default because ordinary Git removal preserves referenced history and repository-wide stashes. It refuses unsafe worktree state unless explicitly forced; `--keep-worktree` performs membership-only removal, and main worktrees, unresolved identities, and common repositories remain protected even under force.
