# Design the Worktree Inspector

Type: grilling
Status: resolved

## Question

How should mntwork accept one or more worktree paths, resolve each worktree root and linked common Git directory through Git plumbing, reject invalid or duplicate membership, compute stable cross-repository identities, and produce the exact mount plan without assuming or scanning default repository roots?

## Comments

- Example: [Worktree Inspector sample output](../05-worktree-inspection-sample-output.md)
- Decision: [Keep common Git directories outside Workspace Roots](../../../docs/adr/0011-keep-common-git-directories-outside-workspace-roots.md)
- Decision: [Serialize shared repository authority across Workspaces](../../../docs/adr/0012-serialize-shared-repository-authority.md)
- Decision: [Use location identities with continuity witnesses](../../../docs/adr/0013-use-location-identities-with-continuity-witnesses.md)

## Answer

The Worktree Inspector is a read-only, explicit-input boundary. It accepts one or more existing directories, including a worktree root or any directory beneath it, but never scans default repository roots or recursively discovers nested repositories. It resolves every input independently, validates the complete batch, and returns one versioned deterministic Inspection Result. No Workspace mutation or executable Logical Mount Grant Set exists unless every candidate is valid.

### Git resolution and structural validation

Inspection clears inherited Git directory, worktree, object, alternate-object, and configuration overrides, disables optional locks, and invokes Git with the submitted directory as its explicit context. Git plumbing must establish that the path is inside a non-bare worktree and return the absolute worktree root, per-worktree Git directory, and Common Git Directory. The Inspector physically canonicalizes those paths through opened directory handles where the platform permits, without case-folding or Unicode normalization.

The resolved worktree must match exactly one record from the repository's NUL-delimited `git worktree list --porcelain -z` output. Its per-worktree Git directory and registration backlink must agree with the resolved root and common directory. Missing, stale, contradictory, bare, or unregistered structures fail. Main worktrees, linked worktrees, nested repositories, and submodule worktrees are structurally eligible when explicitly named and all other rules pass.

Dirty files, detached HEAD, unborn branches, and locked worktrees remain valid observations. They produce stable diagnostics but do not block membership; destructive operations apply their own stronger safety checks. A submitted symlink alias is retained only as provenance, emits `notice[path-canonicalized]`, and resolves to the canonical physical path used for identity and grants. mntwork does not mount arbitrary alias-parent paths merely to preserve input spelling.

### Membership and path safety

Equivalent relative, nested, symlinked, or repeated inputs resolving to one Worktree Identity are duplicate membership and fail the whole batch. Distinct Worktree Identities sharing one Repository Identity are valid, so one Workspace may contain multiple linked worktrees from one repository. Their shared common-directory grant is deduplicated while retaining every consuming Worktree Identity.

Every Common Git Directory resolved for a registered member must be path-disjoint from the Workspace Root: neither canonical path may contain the other. A Worktree Context may equal or descend from the Root, or remain fully external, but may not contain the Root. Conventional main worktrees are therefore valid only as external members. This protects registered repository-wide history and metadata from Root deletion, not uncommitted worktree content or unknown nested repositories.

Inspection is batch-atomic and reports all detected candidate failures in one pass. One missing path, duplicate identity, path collision, or invalid Git structure prevents all membership changes. Canonical containment uses path components, never string prefixes.

### Stable location identities

The Workspace Index owns a non-secret random 128-bit Host Scope ID. Repository and Worktree Identities are host-local, full SHA-256 hashes over explicitly framed bytes:

```text
frame(x) = unsigned 64-bit big-endian byte length || x

repository_payload =
    frame("mntwork.identity") ||
    frame("repository") ||
    frame("1") ||
    frame(Host Scope ID raw 16 bytes) ||
    frame("unix-bytes-v1") ||
    frame(canonical Common Git Directory bytes)

repository_id =
    "repository:v1:sha256:" || lowercase_hex(SHA256(repository_payload))

worktree_payload =
    frame("mntwork.identity") ||
    frame("worktree") ||
    frame("1") ||
    frame(Host Scope ID raw 16 bytes) ||
    frame(repository_id ASCII bytes) ||
    frame("unix-bytes-v1") ||
    frame(canonical worktree-root bytes)

worktree_id =
    "worktree:v1:sha256:" || lowercase_hex(SHA256(worktree_payload))
```

Paths use lossless native Unix bytes. Persisted and JSON representations encode those bytes losslessly, while separately escaped display text is never used for hashing, containment, mounting, or reinspection. Input order, submitted alias, branch, commit, remote, inode, and device number do not participate. Full lowercase 64-digit digests are required; unknown, truncated, uppercase, imported, or tampered IDs are rejected or recomputed. Migration must atomically cross-index lease keys across identity versions, and no active Sandbox may span migration.

Moving a repository or worktree changes identity and requires explicit relinking. Recreating a different repository at the same canonical path retains the location identity because Git exposes no immutable repository UUID. A separately persisted Continuity Witness therefore records the strongest available filesystem, volume, per-worktree Git directory, common-directory, and registration-backlink evidence. Reinspection detects ordinary replacement and drift, but does not promise impossible guaranteed detection when replacement reproduces all available evidence. Inspector never writes enrollment markers into Git repositories.

### Live equivalence and repository authority

Continuity Witness answers whether one stable location still resembles its recorded incarnation. Separate temporary Live Repository and Worktree Object Keys answer whether different paths currently expose the same repository authority or worktree instance. These handle-derived keys never become durable identity.

One canonical Host Scope Lease Manager compares every candidate with every active or reserved Repository Lease and atomically reserves the complete live repository-authority equivalence set before Sandbox preparation. This closes the race where two processes inspect different aliases and acquire different path-derived lease keys simultaneously. Reservations survive through Sandbox lifetime and release only after Runtime Observation proves absence. Historical alias records locate comparison candidates but never replace live evidence.

Weak, missing, contradictory, remounted, or mount-namespace-incomparable evidence tied to active authority fails closed within the relevant uncertainty domain. Explicit per-Workspace shared-repository policy may permit concurrent writable authority across the complete equivalence set; `--force` never bypasses it. The host account and mntwork process environment are trusted: mntwork rechecks handles and evidence immediately before mounting and verifies targets afterward, but does not claim protection from a malicious same-user host process racing path replacement.

### Logical mount grants

Inspector emits the exact Logical Mount Grant Set; only Runtime Driver preparation chooses a physical mount plan. The set contains purpose-distinct read-write identical-path grants for Workspace Root, every canonical worktree root, and every unique Common Git Directory. A stable grant ID derives from the complete normalized grant tuple, and shared grants retain sorted consumer identities.

Grants deduplicate only when purpose, source, target, access, and identical-path requirements all match. Containment alone never removes a logical grant. A Runtime Driver may map several logical grants to one physical mount only after proving every target resolves to the intended host object, path spellings remain usable, access is neither weaker nor broader, mounts do not shadow targets, and Runtime Observation can verify the realized mapping.

### Submodules, diagnostics, and reinspection

Submodules and independent nested repositories never become members through recursion. Inspector may read root `.gitmodules` as bounded declared metadata without initializing, cloning, networking, or mutation. Submodule presence emits a notice; embedded Git data inside Workspace Root or missing initialized metadata emits a warning. Absorbed submodule metadata under a registered parent's external Common Git Directory works through the parent's grants. An explicitly supplied submodule is inspected as its own candidate.

Successful unusual states emit stable notices or warnings on stderr and remain exit zero. JSON includes the same structured diagnostics with severity, stable code, affected identity, and details; stderr is never automation's sole record. Invalid batches return candidate diagnostics but no executable grant or lease evidence.

Every Sandbox preparation and destructive operation performs fresh inspection. Identity, canonical-path, witness, registration, or live-object drift produces stale/conflict and never updates membership automatically. Missing paths do not prove deletion; explicit relink, replace, remove, or reconciliation remains required.

An opt-in UUID sentinel write probe may support `doctor` diagnostics and Runtime Driver conformance tests against disposable fixtures. Positive visibility through another path can corroborate shared writable storage, but absence is inconclusive; the probe never participates in normal inspection, identity, or Repository Lease authority.

The Inspection Result contains schema version, validity, Host Scope ID, canonical Workspace Root, deterministic worktree records, identities, Git facts, persistent Continuity Witnesses, submodule summaries, structured diagnostics, the Logical Mount Grant Set, and transaction-scoped live lease evidence. Canonical-path ordering makes output independent of input order. Human output and stderr derive from the same result as versioned JSON.
