# Worktree Inspector sample output

These examples are illustrative review material, not approved schema. IDs, digests, timestamps, and filesystem evidence are invented but shaped like proposed values.

## Successful human output

Command:

```sh
mntwork ws init /Users/yanni/workspaces/payments \
  --member /Users/yanni/aliases/api \
  --member /Users/yanni/workspaces/payments/api-418
```

stdout:

```text
Workspace Root  /Users/yanni/workspaces/payments
Inspection      valid

Worktree Contexts
  /Volumes/Dev/worktrees/api-417
    kind        linked
    branch      feature/417
    repository  repository:v1:sha256:111111111111…
    worktree    worktree:v1:sha256:aaaaaaaaaaaa…
    state       dirty

  /Users/yanni/workspaces/payments/api-418
    kind        linked
    branch      feature/418
    repository  repository:v1:sha256:111111111111…
    worktree    worktree:v1:sha256:bbbbbbbbbbbb…
    state       clean

Logical Mount Grants
  workspace-root  /Users/yanni/workspaces/payments
  worktree         /Volumes/Dev/worktrees/api-417
  worktree         /Users/yanni/workspaces/payments/api-418
  common-git       /Users/yanni/Git-Repositories/bare/api.git

2 worktrees, 1 repository, 4 logical grants
```

stderr:

```text
notice[path-canonicalized]: /Users/yanni/aliases/api resolves to /Volumes/Dev/worktrees/api-417
warning[worktree-dirty]: /Volumes/Dev/worktrees/api-417 contains uncommitted changes
notice[submodules-declared]: /Volumes/Dev/worktrees/api-417 declares 2 submodules; neither becomes a Workspace member automatically
```

## Successful JSON output

```json
{
  "schema": "mntwork.worktree-inspection/v1",
  "valid": true,
  "host_scope_id": "7af63cf8-80b8-4b63-a15f-3bd9444e5541",
  "workspace_root": {
    "path": {
      "encoding": "unix-bytes-v1",
      "bytes_base64url": "L1VzZXJzL3lhbm5pL3dvcmtzcGFjZXMvcGF5bWVudHM",
      "display": "/Users/yanni/workspaces/payments"
    }
  },
  "worktrees": [
    {
      "worktree_id": "worktree:v1:sha256:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
      "repository_id": "repository:v1:sha256:1111111111111111111111111111111111111111111111111111111111111111",
      "kind": "linked",
      "submitted_path": {
        "encoding": "unix-bytes-v1",
        "bytes_base64url": "L1VzZXJzL3lhbm5pL2FsaWFzZXMvYXBp",
        "display": "/Users/yanni/aliases/api"
      },
      "canonical_path": {
        "encoding": "unix-bytes-v1",
        "bytes_base64url": "L1ZvbHVtZXMvRGV2L3dvcmt0cmVlcy9hcGktNDE3",
        "display": "/Volumes/Dev/worktrees/api-417"
      },
      "git": {
        "worktree_git_directory": "/Users/yanni/Git-Repositories/bare/api.git/worktrees/api-417",
        "common_git_directory": "/Users/yanni/Git-Repositories/bare/api.git",
        "head": "4174174174174174174174174174174174174174",
        "branch": "refs/heads/feature/417",
        "locked": false,
        "prunable": false,
        "dirty": true
      },
      "continuity_witness": {
        "scheme": "darwin-apfs-v1",
        "worktree_root": "opaque-witness-01",
        "worktree_git_directory": "opaque-witness-02",
        "common_git_directory": "opaque-witness-03",
        "registration_backlink_digest": "sha256:2222222222222222222222222222222222222222222222222222222222222222",
        "assurance": "strong"
      },
      "submodules": {
        "declared": 2,
        "initialized": 1,
        "embedded_git_directories": 0
      },
      "diagnostics": [
        {
          "severity": "notice",
          "code": "path-canonicalized",
          "message": "submitted path resolves to a different canonical path"
        },
        {
          "severity": "warning",
          "code": "worktree-dirty",
          "message": "worktree contains uncommitted changes"
        },
        {
          "severity": "notice",
          "code": "submodules-declared",
          "message": "2 declared submodules do not become Workspace members automatically"
        }
      ]
    },
    {
      "worktree_id": "worktree:v1:sha256:bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb",
      "repository_id": "repository:v1:sha256:1111111111111111111111111111111111111111111111111111111111111111",
      "kind": "linked",
      "submitted_path": {
        "encoding": "unix-bytes-v1",
        "bytes_base64url": "L1VzZXJzL3lhbm5pL3dvcmtzcGFjZXMvcGF5bWVudHMvYXBpLTQxOA",
        "display": "/Users/yanni/workspaces/payments/api-418"
      },
      "canonical_path": {
        "encoding": "unix-bytes-v1",
        "bytes_base64url": "L1VzZXJzL3lhbm5pL3dvcmtzcGFjZXMvcGF5bWVudHMvYXBpLTQxOA",
        "display": "/Users/yanni/workspaces/payments/api-418"
      },
      "git": {
        "worktree_git_directory": "/Users/yanni/Git-Repositories/bare/api.git/worktrees/api-418",
        "common_git_directory": "/Users/yanni/Git-Repositories/bare/api.git",
        "head": "4184184184184184184184184184184184184184",
        "branch": "refs/heads/feature/418",
        "locked": false,
        "prunable": false,
        "dirty": false
      },
      "continuity_witness": {
        "scheme": "darwin-apfs-v1",
        "worktree_root": "opaque-witness-04",
        "worktree_git_directory": "opaque-witness-05",
        "common_git_directory": "opaque-witness-03",
        "registration_backlink_digest": "sha256:3333333333333333333333333333333333333333333333333333333333333333",
        "assurance": "strong"
      },
      "submodules": {
        "declared": 0,
        "initialized": 0,
        "embedded_git_directories": 0
      },
      "diagnostics": []
    }
  ],
  "logical_mount_grant_set": {
    "grants": [
      {
        "grant_id": "mount-grant:v1:sha256:4444444444444444444444444444444444444444444444444444444444444444",
        "purpose": "workspace-root",
        "source": "/Users/yanni/workspaces/payments",
        "target": "/Users/yanni/workspaces/payments",
        "access": "read-write",
        "identical_path_required": true,
        "consumers": []
      },
      {
        "grant_id": "mount-grant:v1:sha256:5555555555555555555555555555555555555555555555555555555555555555",
        "purpose": "worktree",
        "source": "/Volumes/Dev/worktrees/api-417",
        "target": "/Volumes/Dev/worktrees/api-417",
        "access": "read-write",
        "identical_path_required": true,
        "consumers": [
          "worktree:v1:sha256:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
        ]
      },
      {
        "grant_id": "mount-grant:v1:sha256:6666666666666666666666666666666666666666666666666666666666666666",
        "purpose": "worktree",
        "source": "/Users/yanni/workspaces/payments/api-418",
        "target": "/Users/yanni/workspaces/payments/api-418",
        "access": "read-write",
        "identical_path_required": true,
        "consumers": [
          "worktree:v1:sha256:bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
        ]
      },
      {
        "grant_id": "mount-grant:v1:sha256:7777777777777777777777777777777777777777777777777777777777777777",
        "purpose": "common-git",
        "source": "/Users/yanni/Git-Repositories/bare/api.git",
        "target": "/Users/yanni/Git-Repositories/bare/api.git",
        "access": "read-write",
        "identical_path_required": true,
        "consumers": [
          "worktree:v1:sha256:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
          "worktree:v1:sha256:bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
        ]
      }
    ]
  },
  "lease_evidence": {
    "lifetime": "lease-transaction-only",
    "repository_equivalence_sets": [
      {
        "live_repository_object_key": "darwin-apfs-v1:opaque-live-key-01",
        "assurance": "strong",
        "repository_ids": [
          "repository:v1:sha256:1111111111111111111111111111111111111111111111111111111111111111"
        ]
      }
    ],
    "worktrees": [
      {
        "worktree_id": "worktree:v1:sha256:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
        "live_worktree_object_key": "darwin-apfs-v1:opaque-live-worktree-01"
      },
      {
        "worktree_id": "worktree:v1:sha256:bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb",
        "live_worktree_object_key": "darwin-apfs-v1:opaque-live-worktree-02"
      }
    ]
  },
  "diagnostics": [
    {
      "severity": "notice",
      "code": "shared-repository",
      "message": "2 Worktree Contexts share one Repository Identity"
    }
  ]
}
```

## Invalid batch

Command:

```sh
mntwork ws member-add \
  /Volumes/Dev/worktrees/api-417 \
  /missing/api-419 \
  /Volumes/Dev/worktrees/api-417/src
```

stderr:

```text
error[path-not-found]: /missing/api-419 does not exist
error[duplicate-worktree]: /Volumes/Dev/worktrees/api-417/src resolves to the same Worktree Identity as /Volumes/Dev/worktrees/api-417
```

JSON:

```json
{
  "schema": "mntwork.worktree-inspection/v1",
  "valid": false,
  "candidates": [
    {
      "submitted_path": "/Volumes/Dev/worktrees/api-417",
      "status": "valid",
      "worktree_id": "worktree:v1:sha256:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    },
    {
      "submitted_path": "/missing/api-419",
      "status": "invalid",
      "diagnostics": [
        {
          "severity": "error",
          "code": "path-not-found"
        }
      ]
    },
    {
      "submitted_path": "/Volumes/Dev/worktrees/api-417/src",
      "status": "invalid",
      "diagnostics": [
        {
          "severity": "error",
          "code": "duplicate-worktree",
          "duplicates_candidate": 0
        }
      ]
    }
  ],
  "logical_mount_grant_set": null,
  "lease_evidence": null
}
```

No Workspace Definition, membership, lease, or Sandbox resource changes when `valid` is false.
