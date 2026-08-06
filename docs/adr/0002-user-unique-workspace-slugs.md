# Keep Workspace slugs unique per user

Each Workspace receives a stable opaque Workspace ID and a human-readable Workspace Slug. Slugs are unique within the current user's Workspace Index, including while a Sandbox Environment is stopped, so CLI references and `.localhost` routes remain unambiguous without imposing machine-wide or multi-user coordination.
