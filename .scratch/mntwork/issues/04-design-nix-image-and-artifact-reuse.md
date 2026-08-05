# Design the Nix image and artifact reuse model

Type: research
Status: open

## Question

How should mntwork build and version a minimal trusted Nix image with flakes, `direnv`, and `nix-direnv`, authorize each mounted worktree for immediate evaluation, and reuse Nix artifacts across Workspaces without exposing the host Nix installation or making correctness depend on a mutable shared cache, including when the configured data root lives on an external drive?
