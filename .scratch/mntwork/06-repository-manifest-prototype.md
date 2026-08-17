# Repository Manifest / Workspace Definition — prototype

> **PROTOTYPE — throwaway.** A rough take to react to for ticket
> [06 — Define Repository Manifest and Workspace Definition contracts](issues/06-define-repository-manifest-and-workspace-definition.md).
> Nothing here is decided. Field names, defaults, and stances are deliberately
> concrete so they can be argued with, then discarded.

## Worked scenario

One Workspace, `acme-checkout`, with two Worktree Contexts from different repositories:

| Member | Worktree Context | Repository |
| --- | --- | --- |
| Root | `~/work/acme-checkout` | (Workspace Root, not a repo) |
| A | `~/work/acme-checkout/api` | `acme/api` |
| B | `~/src/acme-web-worktrees/checkout` | `acme/web` (external member) |

Both repositories ship a Repository Manifest. They conflict.

---

## 1. Repository Manifest — `.mntwork.toml`

Committed at the root of each repository's worktree. Optional. Read-only input.

`acme/api` → `api/.mntwork.toml`:

```toml
manifest_version = 1

[sandbox]
memory_min = "4Gi"
cpus_min = 2

[sandbox.capabilities]
required = ["workload-engine"]     # api integration tests need in-sandbox Docker
preferred = ["nested-virt"]

[[ports]]
name = "api"
container_port = 8080
route = "api"                       # -> api.acme-checkout.localhost

[nix]
envrc = ".envrc"                    # relative to this worktree root
```

`acme/web` → `checkout/.mntwork.toml`:

```toml
manifest_version = 1

[sandbox]
memory_min = "8Gi"                  # CONFLICT with api's 4Gi
cpus_min = 2

[sandbox.capabilities]
required = []
preferred = ["nested-virt"]

[[ports]]
name = "web"
container_port = 3000
route = "web"

[[ports]]
name = "storybook"
container_port = 6006
route = "api"                       # CONFLICT: same route name as api's port
```

### Deliberately absent from the manifest

These are proposed as **not expressible by a repository**, each for a stated reason:

| Field | Why not | Source of truth |
| --- | --- | --- |
| `runtime` | ADR 0004 fixes the precedence chain and does not include manifests | CLI / Definition / user-global |
| `data_root` | Host storage placement; a repo cannot know about your external drive | user-global config |
| Nix GC age / threshold | Host storage policy, not project intent | user-global config |
| Shared Artifact Cache endpoint + keys | Trust boundary (ADR 0009). A repo naming its own substituter is a supply-chain hole | user-global config |
| Mount grants for host paths | ADR 0011 / 0012 boundary. A repo asking to mount `~/.ssh` must never be honoured | CLI only |
| Credential grants | Ticket 07, but same reasoning | CLI only |
| Workspace slug / ID | ADR 0002 — user-scoped uniqueness | generated |

The shape of that table *is* the proposal: manifests describe **what the project
needs to run**, never **how this host is arranged or what it trusts**.

---

## 2. Conflict, rendered three ways — **(c) chosen**

Both manifests want `memory_min`; both want the route name `api`.

### Stance (a) — hard error

```
$ mntwork workspace member-add ~/src/acme-web-worktrees/checkout

error: repository manifests conflict
  sandbox.memory_min
    4Gi   from acme/api      (~/work/acme-checkout/api/.mntwork.toml)
    8Gi   from acme/web      (~/src/acme-web-worktrees/checkout/.mntwork.toml)
  ports.route "api"
    api:8080        from acme/api
    storybook:6006  from acme/web

resolve explicitly, then retry:
  --set sandbox.memory_min=8Gi
  --set ports.storybook.route=storybook
```

Nothing is written. `workspace.toml` gains a `[overrides]` block recording the
chosen values and that they were user-supplied.

### Stance (b) — last-repo-wins by membership order

```
$ mntwork workspace member-add ~/src/acme-web-worktrees/checkout
warning: acme/web overrode sandbox.memory_min 4Gi -> 8Gi (from acme/api)
warning: acme/web overrode route "api" -> storybook:6006 (from acme/api)
added member acme/web
```

Silent-ish. Reordering members changes the Sandbox. `api`'s route is now stolen
and its dev finds out at runtime.

### Stance (c) — manifests request, Definition decides

Numeric requirements take the **maximum** (a request is a floor, not a value);
capability sets **union**; route names must be unique or the request is refused —
i.e. resolution is per-field-semantics, not per-field-precedence.

```
$ mntwork workspace member-add ~/src/acme-web-worktrees/checkout
note: sandbox.memory_min resolved to 8Gi (max of 4Gi acme/api, 8Gi acme/web)
error: route name "api" requested by both acme/api and acme/web
  route names are Workspace-unique (ADR 0002); resolve with:
    --set ports.storybook.route=storybook
```

Note what (c) exposes: `memory_min` has an obvious merge rule and needs no human,
while `route` genuinely does not. Stance (c) says the *field kind* decides, so
only true conflicts reach the user.

### 2a. Merge rules — the cost of (c)

Choosing (c) means the schema is not just a field list; **every field carries a
declared merge rule**, and a field without one cannot ship.

| Rule | Semantics | Fields |
| --- | --- | --- |
| `floor` | Take the maximum. A request is a lower bound, never an exact value. | `memory_min`, `cpus_min` |
| `union` | Set union; duplicates collapse. | `capabilities.preferred` |
| `union-strict` | Set union, but any member unsatisfiable on this host fails preparation for the whole Workspace | `capabilities.required` |
| `unique` | Values must not collide; collision is refused with a `--set` remedy | `ports.*.route`, `ports.*.name` |
| `per-member` | Never merged; scoped to its own Worktree Context | `nix.envrc`, `ports.*.container_port` |

### 2b. Two consequences — both settled

**`union-strict` lets one repo veto the Workspace — accepted.** If `acme/api`
requires `workload-engine` and this host's driver reports it `unsupported`, the
whole Workspace refuses to prepare, even for someone working only in `acme/web`.
Chosen over degrade-and-warn because dropping a stated requirement contradicts
ADR 0008 ("security properties never downgrade implicitly"), and over
per-member scoping because a partially-functional Workspace would be a fifth
Sandbox State that ADR 0006's four (absent/stopped/running/stale) don't cover.

**`floor` gets a user-global ceiling — accepted.** Manifests merge freely below
a host-set ceiling. Above it, preparation stops and asks; a non-interactive
invocation must be able to pre-authorize.

```toml
# user-global config, NOT the manifest and NOT the Definition
[limits]
memory_max = "16Gi"
cpus_max = 8
```

```
$ mntwork sandbox start
error: manifest request exceeds user ceiling
  sandbox.memory_min  64Gi requested by acme/api  (ceiling 16Gi)
authorize once with --accept-resource-request, or raise [limits].memory_max
```

#### Open naming problem: the escape hatch must not be `--force`

The obvious spelling is `--force`, but this repo has already given that flag a
narrow, load-bearing meaning. ADR 0005: `--force` "bypasses content-safety
checks only — it cannot expand targets, escape validated paths, or delete common
repositories." ADR 0007 and ADR 0012 lean on the same restraint, and ADR 0012 is
explicit that "force flags never bypass" the repository lease.

Letting `--force` also raise a resource ceiling makes it mean "and also grant
things you didn't ask for", which is precisely the expansion those ADRs forbid.
Proposed instead: a purpose-named flag such as `--accept-resource-request`, so
each authorization stays legible in shell history and CI logs.

#### Open problem: "ask interactively" needs a non-TTY answer

Ticket 01 decided read commands and dry runs emit versioned JSON and that "exit
codes remain stable for automation" — so an interactive prompt cannot be the
only path. Proposed: when stdin is not a TTY, never prompt; fail immediately
with a distinct exit code and a machine-readable reason naming the exact flag
that would authorize it. Agents and CI get a deterministic failure, not a hang.

---

## 3. Workspace Definition — `$WORKSPACE_ROOT/.mntwork/workspace.toml`

Mechanically generated (`CONTEXT.md:175`). Visible, diffable, committable.
**Never hand-edited** — every value traces to a source.

```toml
definition_version = 1

[workspace]
id = "wsp_01JBQ7M2X9K4"            # opaque, stable (ADR 0002)
slug = "acme-checkout"
root = "~/work/acme-checkout"

[preferences]
runtime = "auto"                    # portable; ADR 0004 layer 2

[[members]]
name = "api"
path = "~/work/acme-checkout/api"
relation = "within-root"
manifest = ".mntwork.toml"
manifest_hash = "sha256:1a2b…"      # renders manifest drift detectable

[[members]]
name = "checkout"
path = "~/src/acme-web-worktrees/checkout"
relation = "external"
manifest = ".mntwork.toml"
manifest_hash = "sha256:9f8e…"

[resolved.sandbox]
memory_min = "8Gi"
cpus_min = 2
source = { memory_min = "manifest:checkout", cpus_min = "manifest:api+checkout" }

[resolved.sandbox.capabilities]
required = ["workload-engine"]
preferred = ["nested-virt"]

[[resolved.ports]]
name = "api"
container_port = 8080
route = "api"
source = "manifest:api"

[[resolved.ports]]
name = "storybook"
container_port = 6006
route = "storybook"
source = "override"                 # user resolved the clash

[overrides]                         # the only user-intent surface; CLI-written
"ports.storybook.route" = "storybook"
```

The `source` annotations are the load-bearing idea: a generated file is only
trustworthy if you can see *why* each value is what it is.

## 4. What does NOT live in the Definition

**Workspace State** (runtime-managed, mutable, not committable) — proposed
`$DATA_ROOT/workspaces/<id>/state.toml`:

```toml
resolved_runtime = "apple-container"     # ADR 0004: recorded here, not Definition
sandbox_generation = "gen_7"
base_image_digest = "sha256:c0ffee…"     # ticket 04
image_lock_hash = "sha256:dead…"
nix_store_volume = "mntwork-wsp_01JBQ7M2X9K4-nix"
repository_lease = ["repo_id_a", "repo_id_b"]   # ADR 0012
published_ports = [{ name = "api", host_port = 49812 }]
```

**Workspace Index** (user-scoped registry) — proposed
`$XDG_CONFIG_HOME/mntwork/index.toml`: slug → id → root path, plus Host Scope ID.
Slug uniqueness (ADR 0002) is enforced here, and nowhere else.

Boundary rule proposed: **Definition = intent, State = allocation, Index = location.**
If removing the Sandbox invalidates a value, it belongs in State.

---

## 5. Flake-derived configuration — proposed answer: no

Reading config out of `flake.nix` requires evaluating the flake; evaluation
requires a running Sandbox with an authorized `.envrc` (ADR 0010); building that
Sandbox requires the config. Circular.

`.mntwork.toml` is static TOML precisely so composition happens before any
sandbox exists and before any repository code is executed. A repository that
wants flake-driven values duplicates them into `.mntwork.toml`, and drift is
caught by `manifest_hash`.

---

## Open questions for the human

1. Which conflict stance? (a) hard error, (b) last-repo-wins, (c) per-field-semantics.
2. Does the "deliberately absent" table draw the trust line in the right place?
3. Is `source`-annotating every resolved value worth the file noise?
4. Is `manifest_hash` drift a hard block on `sandbox start`, or a warning?
5. Should `workspace.toml` be committable at all, or always ignored?
