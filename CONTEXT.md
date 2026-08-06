# mntwork

mntwork isolates autonomous agent activity around a named unit of work while preserving the Git worktrees that participate in it.

## Language

**Workspace**:
A named unit of work, identified by a stable slug, that groups one or more Worktree Contexts under one lifecycle.
_Avoid_: Project, worktree, session

**Workspace ID**:
An opaque, stable identity assigned to a Workspace independently of its name or location.
_Avoid_: Workspace slug, path

**Workspace Slug**:
The human-readable name of a Workspace, unique within one user's mntwork registry.
_Avoid_: Workspace ID, directory name

**Workspace Index**:
The user-scoped registry through which mntwork locates and disambiguates known Workspaces.
_Avoid_: System registry, Workspace Definition

**Workspace Root**:
A user-chosen directory that anchors discovery of one Workspace and may contain some or all of its Worktree Contexts.
_Avoid_: Repository root, data root

**Worktree Context**:
A Git worktree, together with its linked common repository, participating in a Workspace.
_Avoid_: Repository, checkout, project

**Sandbox Environment**:
The running isolated environment provisioned for one Workspace.
_Avoid_: Container, VM, workspace

**Sandbox State**:
The observed lifecycle condition of a Workspace's Sandbox Environment: absent, stopped, running, or stale when recorded and runtime reality disagree.
_Avoid_: Workspace state, status

**Repository Manifest**:
Optional versioned defaults contributed by one repository to any Workspace containing its Worktree Context.
_Avoid_: Workspace config, runtime state

**Workspace Definition**:
A visible, mechanically generated description of a Workspace's identity and Worktree Context membership.
_Avoid_: Workspace manifest, runtime state

**Workspace State**:
Mutable runtime-managed facts about a Workspace, including resource identities and allocations.
_Avoid_: Workspace definition, user config
