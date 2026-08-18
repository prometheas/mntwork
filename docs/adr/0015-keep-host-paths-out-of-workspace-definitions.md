# Keep host paths out of the Workspace Definition

Workspace Definition holds intent, Workspace State holds allocation, and the Workspace Index holds location. The Definition therefore records member names, resolved requests with their sources, runtime preference, and explicit overrides, while every host path—including the binding from a member name to its local Worktree Context—lives in the user-scoped Index alongside slug uniqueness and the Host Scope ID.

This makes the mechanically generated Definition genuinely portable and committable, realizing the portability ADR 0004 signalled with `runtime = "auto"`, at the cost of an explicit bind step before a cloned Definition can first start. An unbound member fails preparation rather than resolving to an inferred path, because guessing would reintroduce the repository scanning the Worktree Inspector deliberately excluded.
