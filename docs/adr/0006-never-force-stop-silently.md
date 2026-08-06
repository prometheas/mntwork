# Never force-stop a Sandbox silently

`sandbox stop` requests graceful shutdown and returns failure when its configurable timeout expires instead of escalating automatically. Hard termination requires explicit `--force`; this preserves user intent and reduces loss from interrupting autonomous agents, while accepting that failed graceful stops need a second command.

Workspace membership mutations also refuse while the Sandbox is running or transitioning. They never stop autonomous work implicitly; after an explicit stop, membership changes update the Workspace Definition and the next start recreates runtime mount configuration while retaining reusable storage.
