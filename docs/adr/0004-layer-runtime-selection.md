# Layer runtime selection

Runtime selection follows explicit CLI `--runtime`/`-r`, then the Workspace Definition preference, then user-global auto policy, and finally built-in capability detection. Workspace Definitions may remain portable with `runtime = "auto"`, while each user controls auto preference for their host; the resolved Runtime Driver is recorded in mutable Workspace State, and changing it for an existing Sandbox Environment requires explicit recreation.
