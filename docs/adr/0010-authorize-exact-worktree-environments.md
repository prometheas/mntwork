# Authorize exact worktree environments

Environment Authorization binds one mounted Worktree Context's resolved `.envrc` path and content hash inside one Workspace. Changed files require renewed approval and managed nix-direnv fallback is disabled, trading automatic convenience for protection against silently executing agent-modified shell code or reusing an environment that no longer matches the checkout.
