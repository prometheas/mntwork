# Design the Worktree Inspector

Type: grilling
Status: open

## Question

How should mntwork accept one or more worktree paths, resolve each worktree root and linked common Git directory through Git plumbing, reject invalid or duplicate membership, compute stable cross-repository identities, and produce the exact mount plan without assuming or scanning default repository roots?

