# Define the Runtime Driver interface

Type: grilling
Status: resolved

## Question

What capability-based interface must every Runtime Driver implement to create, inspect, enter, stop, and destroy a persistent Sandbox Environment while isolating host state, mounting multiple Worktree Contexts at identical absolute paths, exposing project container workloads, and reporting unsupported host/runtime combinations before side effects?

## Comments

- Research: [Runtime Driver interface primary-source constraints](../research/runtime-driver-interface-primary-source-facts.md)
- Research: [Host 1Password for Git from a Sandbox Environment](../research/onepassword-git-signing-channel-facts.md)
- Decision: [Keep Runtime Drivers capability-oriented](../../../docs/adr/0008-capability-oriented-runtime-drivers.md)

## Answer

The Runtime Driver is the provider-specific boundary that realizes a Sandbox Environment for mntwork core. Core owns Workspace policy, composition, precedence, and user intent; a driver hides Apple `container`, Docker, Podman, or later provider mechanisms while reporting capability differences honestly. The names below describe semantic operations, not fixed implementation function names.

### Requirements, capabilities, and preparation

Core supplies versioned Sandbox Requirements containing required and preferred capabilities and resources. A driver first performs two read-only stages:

1. **Probe** reports CLI-host and runtime-host identity, driver/backend versions, installation, reachability, service health, and general capabilities.
2. **Prepare** validates the complete Sandbox Requirements—including mounts, storage, workload engine, networking, routes, and channels—and returns either a Prepared Sandbox Plan or normalized incompatibilities.

Capabilities report `supported`, `conditional`, or `unsupported`, with stable identifiers, reasons, remediation, current evidence, and Capability Health (`healthy`, `degraded`, `unhealthy`, or `unknown`). A condition must be satisfied before a required capability qualifies; preferred capabilities may be omitted or degraded only when policy permits, and security properties never downgrade implicitly. Every mutation consumes a prepared plan or equivalent validated intent and rechecks relevant conditions before its first side effect.

Lifecycle and health remain separate: a Sandbox may be `running` while its Workload Engine or a channel is unhealthy. Application-specific health is outside this contract unless later supplied as an explicit requirement.

### Lifecycle operations

Every driver provides these strict semantic operations:

- **Create** realizes an absent Sandbox as stopped and returns one Sandbox Generation plus all driver-owned resource identities. It does not start.
- **Inspect** returns a Runtime Observation: opaque IDs, ownership metadata, lifecycle phase, health, effective capabilities, mounts, bindings, channels, and Owned Resources.
- **Start** transitions an existing stopped Sandbox to running. It does not create.
- **Attach** launches an exact argv inside a running Sandbox, with working directory, user, environment additions or references, stdin/stdout/stderr, interactive/TTY behavior, resize, and cancellation. A shell is a core-selected interactive argv; disconnecting ends the Attachment, not the Sandbox. Process exit and driver/transport failure are distinct.
- **Graceful stop** requests orderly shutdown with a caller-provided timeout and never escalates automatically.
- **Forced stop** performs explicitly authorized hard termination as a separate operation.
- **Destroy** removes only the declared Cleanup Scope from a stopped Sandbox and never stops it implicitly.
- **Reconcile** compares Workspace State with fresh Runtime Observation and returns stale records, missing/duplicate/orphaned resources, and proposed repair or cleanup actions. Read operations do not silently mutate runtime resources.

Core composes these primitives to implement the settled CLI lifecycle. Operations check expected identity/state so concurrent or contradictory changes return conflict. Repetition is safe only when the target result can be proven already achieved.

### Mount and storage boundary

Core computes explicit Mount Grants; drivers never infer membership, omit grants, create missing sources, or broaden access silently. Each grant declares purpose, host source, sandbox target, access mode, and whether identical absolute paths are required.

Workspace Root, every Worktree Context, and every linked common Git directory are required read-write grants at identical absolute paths. External members/common directories remain distinct logical grants. A driver may coalesce overlapping physical mounts only when effective access and visibility remain identical and the Prepared Sandbox Plan reports that mapping. The configurable mntwork data root uses explicit source/target grants and distinguishes persistent from disposable purposes. Any missing, conflicting, or unsupported required grant fails preparation before side effects.

### Workload engine, networking, and channels

Workload Engine capability is independent from the Runtime Driver provider. It supplies a sandbox-scoped Docker or Podman service and reports engine kind, API/version, endpoint, health, storage location, isolation scope, and lifecycle owner without exposing whether the driver used nesting, a sidecar, a VM service, or another mechanism. A shared host Docker/Podman socket never silently satisfies this capability; it requires an explicit privileged Channel Grant.

Runtime Drivers realize Port Publications and return effective host address, allocated port, sandbox port, and protocol. Host loopback is the default; LAN/public binding must be explicit. Route Intent preserves the association between a named service and its publication, but Caddy, DNS, TLS, and friendly-route lifecycle remain outside the Runtime Driver. A possible standalone `caddyshack` CLI is a provisional future seam, not a committed architecture.

Channel Grants are denied by default and declare purpose, direction, endpoint/transport, lifetime, and privilege. Drivers transport approved endpoints and report their health; they do not select keys, retrieve secrets, or own broker policy. The safe target distinguishes narrow Git SSH authentication and Git commit-signing broker channels from broad raw SSH-agent forwarding. Drivers never copy private keys into the Sandbox, silently widen a narrow grant, or persist ephemeral channel endpoints after stop/destroy. Whether host 1Password can realize the narrow broker contract remains a separate research question.

### Identity, reconciliation, errors, and cleanup

Drivers assign opaque IDs and durable ownership metadata to every Owned Resource: Workspace ID, Sandbox Generation, driver kind, and resource role. Workspace State stores returned identifiers but never constructs or parses them. Drivers rediscover resources by proven ownership metadata after state loss; ambiguous resources are reported and never deleted automatically.

Core maps settled Runtime Observations to the existing public Sandbox States `absent`, `stopped`, and `running`; unknown, contradictory, missing, duplicated, or partial reality maps to `stale`. Transitional driver phases remain operation detail rather than new stable Sandbox States.

Driver failures normalize operation, stable category, retryability, Mutation Outcome (`no-effects`, `partial-effects`, or `unknown`), affected capability/resource IDs, safe remediation, and redacted backend diagnostics. Stable categories include unsupported capability, incompatible host/runtime, runtime unavailable, not found, invalid state, conflict, permission denied, timeout, backend failure, and cleanup incomplete. Partial or unknown effects require fresh inspection/reconciliation before another mutation.

Creation failure triggers best-effort rollback. Cleanup is retry-safe and never deletes user-owned Mount Grant sources. Destroy succeeds only after verifying that no in-scope Owned Resources remain; otherwise it returns cleanup incomplete with residual identities and a safe retry path. Intentionally retained persistent resources are reported separately.

### Runtime selection and maintained knowledge

Selection preserves settled precedence: explicit CLI runtime, Workspace preference, user-global auto policy, then built-in detection. Explicit CLI or Workspace choices are hard constraints and fail incompatibly rather than falling back. For `auto`, allowed drivers probe and prepare against the complete Sandbox Requirements; only valid plans qualify. User-global policy ranks qualified drivers, preferred-capability coverage may act as a policy-controlled tie-breaker, and built-in ordering is deterministic. Selection explains rejected candidates. The resolved driver remains in Workspace State and changes only through explicit Sandbox recreation.

A versioned, evidence-backed Capability Catalog records known driver/version/host support and limits; live Capability Reports remain authoritative for the current machine. Unknown catalog data is not unsupported capability, catalog claims require source or conformance evidence, and successful preparation is the final qualification. Catalog schema and maintenance belong to a separate ticket.
