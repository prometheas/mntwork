# Initialize Workspace definitions explicitly

mntwork requires `workspace init [ROOT]` to create `$WORKSPACE_ROOT/.mntwork/workspace.toml`; when an explicit Root does not exist, `init` previews and creates it, while an omitted Root means the existing current directory. Sandbox commands discover that definition upward but never create user-visible files implicitly. This trades a one-time explicit setup step for predictable filesystem mutation, portable discovery, and a stable lifecycle identity across multiple repositories.

The Workspace Root is mounted read-write and becomes the default in-sandbox working directory. Worktree Contexts outside it are mounted separately.
