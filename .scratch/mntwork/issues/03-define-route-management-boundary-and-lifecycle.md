# Define the route-management boundary and lifecycle

Type: grilling
Status: open
Blocked by: 01, 02, 06

## Question

Where should friendly-route, DNS, TLS, and proxy lifecycle live—inside mntwork, in a potentially standalone `caddyshack` CLI, or behind another integration boundary—and what driver-neutral handoff should consume Runtime Driver Port Publications, derive Workspace/service routes, integrate with an existing host Caddy installation, report reload failures, retain routes while a Sandbox Environment lives, and reconcile stale routes after crashes or reboots?

`caddyshack` is a provisional seam to evaluate, not a committed product or architecture.
