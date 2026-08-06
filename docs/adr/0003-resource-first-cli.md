# Use resource-first CLI grammar

mntwork commands use `mntwork <resource> <verb>` so lifecycle and destructive operations name their target explicitly. Workspace commands manage definitions and registration, while sandbox commands manage Sandbox Environments and runtime state; the extra command word is accepted in exchange for safer destructive actions, clearer help, and room for future route and cache resources.

Within the sandbox resource, `start` converges to a running environment without attaching and `shell` converges and opens an interactive shell. `exec` runs an explicit command only against a running environment unless `--start` explicitly permits convergence first. The CLI does not expose a separate ambiguous `enter` verb.

The global `--workspace` selector has the `-w` alias and precedes the resource, following Git-like global option placement. `workspace` and `sandbox` expose the help-visible aliases `ws` and `sb`; documentation uses full resource names while interactive examples may show both forms.
