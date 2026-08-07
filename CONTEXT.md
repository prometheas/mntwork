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

**Runtime Driver**:
A provider-specific boundary through which mntwork manages a Sandbox Environment while exposing the capabilities honestly available on the current host.
_Avoid_: Runtime, backend, provider

**Capability Report**:
A Runtime Driver's current declaration of which sandbox capabilities are supported, conditional, or unsupported, including reasons for unavailable capabilities.
_Avoid_: Feature list, compatibility matrix

**Capability Catalog**:
Versioned, evidence-backed knowledge of Runtime Driver support across known host, runtime, and version combinations, distinct from live Capability Reports.
_Avoid_: Capability Report, runtime probe

**Capability Health**:
The observed healthy, degraded, unhealthy, or unknown condition of a driver-provided capability, independent of Sandbox State.
_Avoid_: Sandbox State, process status

**Sandbox Requirements**:
The required and preferred capabilities and resources a Workspace asks a Runtime Driver to provide before its Sandbox Environment may be created or changed.
_Avoid_: Runtime config, driver options

**Prepared Sandbox Plan**:
A side-effect-free, driver-specific plan that has validated one set of Sandbox Requirements against current host and runtime conditions.
_Avoid_: Sandbox definition, dry run

**Runtime Observation**:
A Runtime Driver's current account of one Sandbox Environment's identity, lifecycle phase, health, capabilities, and owned resources.
_Avoid_: Sandbox State, Workspace State

**Reconciliation**:
Comparison of Workspace State with Runtime Observation to identify stale records, residual resources, and explicit repair or cleanup actions.
_Avoid_: Status refresh, automatic repair

**Sandbox Generation**:
One concrete realization of a Workspace's Sandbox Environment, with an identity that changes whenever the Sandbox is recreated.
_Avoid_: Workspace ID, Runtime Driver ID

**Owned Resource**:
A runtime resource whose metadata proves that a Runtime Driver created it for one Sandbox Generation and identifies its role.
_Avoid_: Named resource, mounted host path

**Cleanup Scope**:
The Owned Resources selected for removal in one operation, distinct from resources intentionally retained and user-owned mount sources.
_Avoid_: Workspace deletion, recursive delete

**Mutation Outcome**:
A Runtime Driver's declaration that a failed operation caused no effects, caused partial effects, or left its effects unknown.
_Avoid_: Exit status, Sandbox State

**Attachment**:
A connection to one process launched inside a running Sandbox Environment whose lifetime does not control the Sandbox Environment.
_Avoid_: Sandbox session, Sandbox Environment

**Mount Grant**:
An explicit allowance for a Sandbox Environment to access one host path at a declared sandbox path and access mode.
_Avoid_: Volume, shared folder

**Workload Engine**:
A sandbox-scoped Docker or Podman service used by projects inside a Sandbox Environment, independently of the Runtime Driver's own provider.
_Avoid_: Runtime Driver, host Docker

**Channel Grant**:
An explicit allowance for narrowly scoped communication between a Sandbox Environment and a host or broker endpoint for one declared purpose.
_Avoid_: Mount Grant, socket pass-through

**Credential Broker**:
A host-side authority that performs an approved authentication or signing operation without exposing private key material to the Sandbox Environment.
_Avoid_: Credential store, secret mount

**Port Publication**:
A request to expose one Sandbox Environment port through a declared host address, port policy, and protocol.
_Avoid_: Route, proxy

**Route Intent**:
A driver-neutral request to associate a named service route with a Port Publication for later realization by a route manager.
_Avoid_: Port Publication, Caddy config

**Repository Manifest**:
Optional versioned defaults contributed by one repository to any Workspace containing its Worktree Context.
_Avoid_: Workspace config, runtime state

**Workspace Definition**:
A visible, mechanically generated description of a Workspace's identity and Worktree Context membership.
_Avoid_: Workspace manifest, runtime state

**Workspace State**:
Mutable runtime-managed facts about a Workspace, including resource identities and allocations.
_Avoid_: Workspace definition, user config
