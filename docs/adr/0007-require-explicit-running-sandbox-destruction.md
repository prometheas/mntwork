# Require explicit destruction of a running Sandbox

Plain `sandbox destroy` refuses any running, transitional, or actively attached Sandbox Environment. `--stop` explicitly requests graceful stop before destruction, while `--force` authorizes hard stop and disconnection; `--yes` alone skips confirmation and `--dry-run` only previews. This rejects convenient implicit teardown because autonomous processes and attached shells make running state a meaningful safety boundary.
