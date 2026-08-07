# Host 1Password for Git from a Sandbox Environment

Accessed: 2026-08-06

Scope: facts and a safe conceptual model for later Runtime Driver contract discussion. This note does not select an implementation, prove any driver's feasibility, or resolve credential-policy tickets.

## Two different uses of an SSH key

### SSH authentication for Git transport

For a remote such as `git@github.com:owner/repo.git`, sandbox `ssh` proves possession of an accepted key while opening the network connection used by `git fetch`, `pull`, or `push`. An SSH agent is a key provider: client asks it to use a private key, but does not read that key. 1Password says its agent makes eligible keys available, asks for consent before use, and keeps private keys inside 1Password ([1Password agent overview](https://www.1password.dev/ssh/agent), [security model](https://www.1password.dev/ssh/agent/security)).

Safe conceptual flow:

1. Sandbox Git invokes sandbox SSH for a named Git host.
2. SSH obtains public-key identities and requests a private-key operation through an approved host channel.
3. Host 1Password identifies the key and asks user to authorize its use according to host policy.
4. Only the signature result returns; private key does not enter sandbox.
5. SSH completes authentication with Git host, then Git traffic uses that SSH connection.

### SSH-format Git commit signing

Commit signing does not authenticate a network connection. Git builds commit payload, requests a detached SSH signature, then embeds it in commit. Git documents `gpg.format=ssh`; default `gpg.ssh.program` is `ssh-keygen` ([Git configuration](https://git-scm.com/docs/git-config)). OpenSSH documents that `ssh-keygen -Y sign` may be given a public key whose private half is available through `ssh-agent`; verification uses `-Y verify`, signer identity, namespace, and allowed-signers file ([ssh-keygen](https://man.openbsd.org/ssh-keygen.1)). Git's signature format is an armored `SSH SIGNATURE` block ([gitformat-signature](https://git-scm.com/docs/gitformat-signature)).

Safe conceptual flow:

1. Sandbox Git prepares exact commit payload and selects SSH signing format and public signing key.
2. Sandbox signing helper requests signature with Git's signing namespace through approved host channel.
3. Host 1Password asks user to authorize chosen key, performs private-key operation, and returns detached signature only.
4. Git embeds signature in commit. No private key or secret value is copied into sandbox.
5. Verification is public-data work: Git/`ssh-keygen` checks commit payload and signature against configured `gpg.ssh.allowedSignersFile`; it does not need 1Password or private key. Git treats a matching public key in that file as trusted for SSH verification ([Git configuration](https://git-scm.com/docs/git-config)). Hosting services separately require public key registration as signing key ([1Password commit-signing guide](https://www.1password.dev/ssh/git-commit-signing)).

Authentication key and signing key may be same mathematical key, but operations, policy, configuration, and audit meaning differ. Runtime contract should not collapse them into one generic "Git credential" permission.

## What raw 1Password SSH-agent access exposes

On macOS, 1Password documents Unix socket at `~/Library/Group Containers/2BUA8C4S2C.com.1password/t/agent.sock`, selectable through `IdentityAgent` or `SSH_AUTH_SOCK`; optional `~/.1password/agent.sock` is only symlink ([1Password setup](https://www.1password.dev/ssh/get-started)). Agent protocol lets clients enumerate available public identities and request supported private-key operations. 1Password retains keys and prompts for consent, but its own forwarding guide warns that approving remote use authorizes entire connected IDE/app and that any process under same remote OS user can then use key ([1Password forwarding](https://www.1password.dev/ssh/agent/forwarding)).

OpenSSH's warning is sharper: process able to reach forwarded agent socket cannot extract key material, but can ask agent to perform authentication-capable key operations ([`ForwardAgent`](https://man.openbsd.org/ssh_config.5)). OpenSSH also notes that non-SSH tools forwarding socket may bypass agent's indication that client is remote ([ssh-agent](https://man.openbsd.org/ssh-agent.1)). Therefore "private key never leaves host" is necessary but not equal to least privilege: compromised sandbox can potentially use approved agent authority as signing/authentication oracle.

## macOS/runtime constraints relevant to contract

- Socket is host filesystem endpoint owned by desktop 1Password process, not portable secret file. A sandbox or VM cannot use host path merely because string is known; Runtime Driver needs supported socket transport/remapping. This is architecture inference, not proof for any proposed driver.
- Socket endpoint can change across host sessions. Apple's `container --ssh` explicitly remaps current `SSH_AUTH_SOCK` to `/run/host-services/ssh-auth.sock` and refreshes source after stop, logout/login, and restart ([Apple container how-to](https://github.com/apple/container/blob/main/docs/how-to.md#mount-your-host-ssh-authentication-socket-in-your-container)). Contract therefore needs liveness checking and stale-channel reconciliation rather than persisting unchecked host socket path.
- SSH configuration can override environment: 1Password says `IdentityAgent` takes precedence over `SSH_AUTH_SOCK` in remote-workstation scenario ([1Password forwarding](https://www.1password.dev/ssh/agent/forwarding)). Sandbox setup must avoid inherited `IdentityAgent` silently bypassing intended endpoint.
- Host GUI authorization remains part of flow. Availability requires unlocked/running host 1Password and user authorization policy; Runtime Driver capability report should describe channel availability, not claim credential use always succeeds.

## Safer contract target: narrow broker, not assumed raw forwarding

Ideal future broker should expose purpose-specific operations instead of full agent socket:

- `git-ssh-auth`: authorize selected host/account/key policy and perform only authentication needed for that Git destination/session; do not allow arbitrary agent identity enumeration or unrelated signing.
- `git-commit-sign`: accept commit-signing payload only, fixed Git namespace, selected public key identity, Workspace/repository context, and explicit user policy; return detached SSH signature only.
- Both: sandbox-specific endpoint, least-privilege OS permissions, bounded lifetime, liveness/health, revocation, request attribution, and audit-friendly purpose metadata.

Raw SSH-agent forwarding can remain a distinct, explicitly broad capability when a driver can support it, but must not masquerade as narrow Git authentication or commit-signing channel. Whether 1Password or available Runtime Drivers can enforce proposed narrowing requires later credential/runtime research.

## Runtime Driver contract implications only

- Model Git SSH authentication and SSH commit signing as distinct requested channel purposes.
- Keep secret/key selection and user authorization policy outside Runtime Driver. Driver transports approved endpoint and reports effective scope, health, and lifecycle.
- Preflight must fail before side effects when requested secure channel transport is unsupported.
- Channel observation should detect missing/stale endpoint; stop/destroy should remove sandbox-side endpoint.
- Capability report must distinguish narrow broker, raw SSH-agent forwarding, and unsupported. Never silently downgrade narrow request to raw socket.
