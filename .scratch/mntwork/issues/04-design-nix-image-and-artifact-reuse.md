# Design the Nix image and artifact reuse model

Type: research
Status: resolved

## Question

How should mntwork build and version a minimal trusted Nix image with flakes, `direnv`, and `nix-direnv`, authorize each mounted worktree for immediate evaluation, and reuse Nix artifacts across Workspaces without exposing the host Nix installation or making correctness depend on a mutable shared cache, including when the configured data root lives on an external drive?

## Comments

- Research: [Nix image and artifact reuse primary-source constraints](../research/nix-image-and-artifact-reuse-primary-source-facts.md)
- Decision: [Separate Nix trust and mutation boundaries](../../../docs/adr/0009-separate-nix-trust-and-mutation-boundaries.md)
- Decision: [Authorize exact worktree environments](../../../docs/adr/0010-authorize-exact-worktree-environments.md)

## Answer

mntwork uses four separate ownership and trust boundaries: an immutable Base Image, one persistent Workspace Nix Store per Workspace, Workspace-owned Environment Authorization, and an optional Shared Artifact Cache. No boundary exposes the host Nix installation, shares writable Nix state between Workspaces, or makes cache availability part of correctness.

### Base image and generation identity

mntwork publishes a closure-minimal OCI Base Image built from a committed, locked flake for each supported platform. The release pins Nix, `direnv`, `nix-direnv`, certificates, shell/init utilities, and bootstrap configuration; project toolchains remain in repository flakes. Startup never installs or updates this tooling.

A human release version aids discovery, but the platform-specific OCI manifest digest is authoritative for execution and reconciliation. Workspace State records the digest and image lock hash used by each Sandbox Generation. A newer release never mutates an existing generation: `start` may report it, while an explicit recreate previews the digest change and retained persistent resources. Policy may reject an obsolete image, but never replaces it silently.

### Persistent private store and shared reuse

Each Workspace owns a private writable Workspace Nix Store. It persists across Sandbox stop, start, destroy, and recreation by default, while remaining independent from Sandbox Generation identity. Explicit Workspace pruning removes it. The Sandbox never mounts host `/nix`, connects to a host Nix daemon, uses another Workspace's store, or mutates a shared store.

Cross-Workspace reuse comes from an optional signed Shared Artifact Cache exposed read-only to Workspaces. A single trusted publisher owns writes, signing, verification, serialization, cleanup, and retention; Workspaces receive only a substituter endpoint or read-only snapshot plus public verification keys. Trusted upstream artifacts may be mirrored globally. Project-specific outputs require explicit trusted publication from an independent build of an exact committed revision and lock; ordinary autonomous Workspaces cannot upload directly into the trusted cache. Publication workflow and provenance are a follow-up design ticket.

Cache lookup reports hit, miss, unavailable, untrusted, or corrupt. A verified hit imports the closure into the private store. A miss or outage falls through to later trusted substituters and normal evaluation/build, affecting speed only. Invalid artifacts are rejected and diagnosed. The cache location, contents, generation, hit result, and GC epoch never contribute to Workspace, Sandbox, image, or worktree identity.

This yields three reuse layers: Base Image bootstrap contents, globally reusable trusted substitutions, and a persistent private store that avoids rebuilding when the same Workspace respawns. Multiple Workspaces from one repository share upstream packages immediately; custom outputs become reusable only after trusted publication.

### Worktree evaluation and authorization

Each Worktree Context owns its mounted `flake.nix`, `flake.lock`, and `.envrc`. Explicit preparation validates all mounts, then evaluates the exact mounted worktree. mntwork previews and authorizes each `.envrc` by exact resolved path and content hash inside Workspace-owned direnv state. Authorization persists across shell detach and Sandbox recreation but never crosses Workspace boundaries.

A missing or changed `.envrc` blocks evaluation until explicitly authorized again. Managed nix-direnv configuration disables fallback to a previously successful environment, so a stale shell can never mask failure of the checked-out revision. Preparation reports each worktree as absent, authorized-current, changed-and-blocked, or evaluation-failed. Once current authorization and evaluation succeed, interactive entry can use the prepared environment immediately.

### Persistence, garbage collection, and external media

Current prepared environments have GC roots. Automatic GC never touches running Workspaces, considers only unrooted paths older than a configured age, and runs only above a configured storage threshold. Shared-cache retention is independent. Manual pruning supports dry-run and exact size/target preview.

The configured data root may place Workspace Nix Stores, direnv state, and Shared Artifact Cache data on external media without changing their logical identities or stable in-sandbox paths. Persisted metadata records a logical data-root identity and ownership marker. Preparation fails before side effects when the expected root is absent, wrong, read-only, or permission-incompatible; mntwork never creates a same-looking fallback on internal storage.

If external media disappears while running, cache loss degrades reuse, while operations requiring missing persistent store or state fail visibly. No write silently redirects elsewhere. Reattachment requires identity, ownership, format, and permission validation. Runtime-specific mount and hot-unplug feasibility remains part of the macOS isolation proof.
