# Separate Nix trust and mutation boundaries

mntwork separates a digest-pinned immutable Base Image, one persistent writable Workspace Nix Store per Workspace, and an optional signed Shared Artifact Cache that Workspaces consume read-only. This avoids host Nix exposure and cross-Workspace mutation while retaining fast Sandbox recreation and trusted artifact reuse; cache misses affect performance, never correctness.
