# Define Repository Manifest and Workspace Definition contracts

Type: prototype
Status: resolved

## Question

What minimal versioned schema should optional repository-local `.mntwork.toml` files contribute, how should CLI inputs and multiple repositories merge those contributions into `$WORKSPACE_ROOT/.mntwork/workspace.toml`, what belongs in Workspace Definition versus Workspace State and the Workspace Index, how should a default cache-based data root and explicit external data root behave, where should Workspace Nix Store GC and Shared Artifact Cache retention policy live, and should any project-flake-derived configuration participate without forcing users to author cross-repository composition manually?

## Comments

- Prototype: [Repository Manifest / Workspace Definition strawman](../06-repository-manifest-prototype.md)
- Decision: [Bound what Repository Manifests may oblige](../../../docs/adr/0014-bound-manifest-requests.md)
- Decision: [Keep host paths out of the Workspace Definition](../../../docs/adr/0015-keep-host-paths-out-of-workspace-definitions.md)
- Decision: [Re-authorize manifest drift only on increased requests](../../../docs/adr/0016-reauthorize-manifest-drift-on-increase.md)

## Answer

A Repository Manifest describes **what a project needs in order to run**. It never
describes **how this host is arranged or what it trusts**. Every contract below
follows from that split.

### Manifest schema and scope

`.mntwork.toml` is optional, committed at a Worktree Context root, versioned by
`manifest_version`, and read-only input. It may express sandbox resource floors,
required and preferred capabilities, port publications with route intents, and
its own `.envrc` location.

It may not express runtime selection (ADR 0004 already fixes that precedence
chain and excludes manifests), data root, Nix GC or cache retention policy,
Shared Artifact Cache endpoints or verification keys, mount grants, credential
grants, or Workspace slug and ID. Each exclusion is a trust boundary: a
repository naming its own substituter or requesting a host mount would defeat
ADR 0009, ADR 0011, and ADR 0012 respectively.

### Merging manifests and CLI input

Conflicts resolve by **field semantics, not field precedence**. Every field
carries a declared merge rule and a field without one cannot ship:

| Rule | Semantics | Fields |
| --- | --- | --- |
| `floor` | maximum; a request is a lower bound, never an exact value | `memory_min`, `cpus_min` |
| `union` | set union | `capabilities.preferred` |
| `union-strict` | set union; any member unsatisfiable on this host fails preparation for the whole Workspace | `capabilities.required` |
| `unique` | collision refused with an explicit `--set` remedy | `ports.*.name`, `ports.*.route` |
| `per-member` | never merged; scoped to its Worktree Context | `nix.envrc`, `ports.*.container_port` |

Only true conflicts reach the user; `memory_min` 4Gi versus 8Gi resolves to 8Gi
with a note, while two members claiming one route name is refused.

One member's unsatisfiable required capability fails the whole Workspace rather
than degrading. Degrade-and-warn would contradict ADR 0008's rule that security
properties never downgrade implicitly, and per-member scoping would introduce a
partially-functional Workspace that ADR 0006's four Sandbox States cannot express.

### Manifest Request Ceiling

Because manifests come from cloned repositories, a user-global policy bounds
what they may oblige a Workspace to provide — both resource floors and
privilege-expanding capabilities. Crossing the ceiling stops preparation and
requires purpose-named authorization (`--accept-resource-request`,
`--accept-capability-request`).

The escape hatch is deliberately **not** `--force`. ADR 0005 confines `--force`
to bypassing content-safety checks and forbids it from expanding targets, and
ADR 0012 states force flags never bypass the repository lease. Raising a ceiling
is expansion, so overloading `--force` would contradict decisions already made.

Allowlist admission is checked before availability, because "you never permitted
this" and "this host cannot provide it" have different remedies and must not be
conflated. When stdin is not a TTY, mntwork never prompts: it fails immediately
with a stable exit code and a machine-readable reason naming the authorizing
flag, preserving ticket 01's automation guarantees.

### Definition, State, and Index

**Definition = intent. State = allocation. Index = location.** If removing the
Sandbox invalidates a value it belongs in State; if it names a place on this
host it belongs in the Index.

The Workspace Definition is mechanically generated, never hand-edited, and holds
member names, resolved requests with `source` annotations, runtime preference,
and the `[overrides]` block that is the only user-intent surface. It carries no
host paths, which makes it genuinely portable and committable — the intent ADR
0004 signalled with `runtime = "auto"`.

The Workspace Index owns every host path, including the binding from member name
to local Worktree Context, plus slug uniqueness and the Host Scope ID. A cloned
Definition therefore requires an explicit bind step before first start; an
unbound member is a hard preparation failure, never an inferred path, because
guessing would reintroduce the repository scanning ticket 05 ruled out.

Workspace State keeps the resolved Runtime Driver, Sandbox Generation, Base
Image digest, store volumes, repository leases, and published host ports.

### Manifest drift

Re-authorization is required only when a changed manifest **asks for more**;
other drift re-resolves with a note, and a decrease still emits a note so a
member silently dropping a capability another depends on stays visible.
"Increase" is defined by reusing the merge-rule table as a partial order.

This departs from ADR 0010's strict-equality rule for `.envrc` deliberately.
That rule is all-or-nothing because `.envrc` is executable code where any diff
can do anything; a manifest is declarative data with a bounded schema, so "more"
is computable. Detecting increase requires the Definition to record the
authorized request set per member, not merely a `manifest_hash`.

### Data root, GC, and retention

The data root defaults under the user's cache directory and may move to an
explicit path including external media, with identity and ownership validated
per ticket 04. It is user-global, never manifest-settable: a repository cannot
know about your external drive, and letting it redirect persistent storage would
be a trust boundary violation rather than a convenience.

Workspace Nix Store GC thresholds and Shared Artifact Cache retention are host
storage policy and likewise live in user-global configuration, independent of
each other as ticket 04 established.

### Flake-derived configuration: no

Reading configuration from `flake.nix` requires evaluating the flake; evaluation
requires a running Sandbox with an authorized `.envrc` (ADR 0010); building that
Sandbox requires the configuration. The dependency is circular.

`.mntwork.toml` is static TOML precisely so composition completes before any
Sandbox exists and before any repository code executes. A project wanting
flake-driven values duplicates them into the manifest, and `manifest_hash`
makes the resulting drift detectable.
