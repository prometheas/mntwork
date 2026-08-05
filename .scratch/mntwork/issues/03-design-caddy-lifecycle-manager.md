# Design the Caddy lifecycle manager

Type: grilling
Status: open
Blocked by: 01, 06

## Question

How should mntwork derive Workspace and service `.localhost` routes, allocate collision-free ports, integrate generated snippets with an existing host Caddy installation, report reload failures, retain routes while a Sandbox Environment lives, and reconcile stale routes after crashes or reboots?
