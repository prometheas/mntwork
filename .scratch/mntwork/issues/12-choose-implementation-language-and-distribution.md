# Choose the implementation language and distribution mechanism

Type: grilling
Status: open

## Question

Now that the CLI grammar, Runtime Driver boundary, Worktree Inspector, and configuration contracts are fixed, which implementation language and distribution mechanism should mntwork adopt, given that it must shell out to Git plumbing and multiple container CLIs, produce a single self-contained binary for macOS on Apple Silicon without requiring a host Nix installation, emit versioned JSON with stable exit codes, and remain buildable for Linux as a design constraint?

## Comments

- Graduated from the map's Fog on resolution of [ticket 06](06-define-repository-manifest-and-workspace-definition.md), which fixed the last of the contracts this choice was waiting on.
