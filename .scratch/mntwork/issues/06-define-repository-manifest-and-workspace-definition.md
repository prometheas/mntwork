# Define Repository Manifest and Workspace Definition contracts

Type: prototype
Status: open

## Question

What minimal versioned schema should optional repository-local `.mntwork.toml` files contribute, how should CLI inputs and multiple repositories merge those contributions into `$WORKSPACE_ROOT/.mntwork/workspace.toml`, what belongs in Workspace Definition versus Workspace State and the Workspace Index, how should a default cache-based data root and explicit external data root behave, where should Workspace Nix Store GC and Shared Artifact Cache retention policy live, and should any project-flake-derived configuration participate without forcing users to author cross-repository composition manually?
