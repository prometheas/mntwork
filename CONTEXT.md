# mntwork

mntwork isolates autonomous agent activity around a named unit of work while preserving the Git worktrees that participate in it.

## Language

**Workspace**:
A named unit of work, identified by a stable slug, that groups one or more Worktree Contexts under one lifecycle.
_Avoid_: Project, worktree, session

**Worktree Context**:
A Git worktree, together with its linked common repository, participating in a Workspace.
_Avoid_: Repository, checkout, project

**Sandbox Environment**:
The running isolated environment provisioned for one Workspace.
_Avoid_: Container, VM, workspace

**Repository Manifest**:
Optional versioned defaults contributed by one repository to any Workspace containing its Worktree Context.
_Avoid_: Workspace config, runtime state

**Workspace Record**:
Mechanically generated host state recording a Workspace's membership and allocated runtime resources.
_Avoid_: Workspace manifest, user config
