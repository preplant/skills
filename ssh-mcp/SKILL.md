---

name: ssh-mcp
description: >
  Use this skill for remote inspection, diagnostics, command execution, file
  transfer, or session management through SSH-MCP. It selects the least
  permissive tool, handles policy and execution failures deliberately, and
  minimizes unsafe commands, rejected calls, output, and SSH round trips.
metadata:
  short-description: Safe and efficient remote work through SSH-MCP.

---

# SSH-MCP

Use the least permissive SSH-MCP tool that can complete the authorized task.
Treat the remote host as production unless the user explicitly establishes a
different safety boundary.

## Configuration And Profiles

SSH-MCP stores multi-host configuration and named profiles in a TOML
configuration file. The default platform locations are:

* Linux: `~/.config/ssh-mcp/config.toml` or
  `$XDG_CONFIG_HOME/ssh-mcp/config.toml`;
* macOS: `~/Library/Application Support/ssh-mcp/config.toml`;
* Windows: `%APPDATA%\ssh-mcp\config.toml`.

Profiles are declared as `[[profiles]]` entries in this file. Do not search
OpenSSH configuration, project repositories, or unrelated MCP configuration
when an SSH-MCP profile needs to be inspected or changed.

An SSH-MCP process may instead be started with `--config <path>`. When an
explicit config path is configured for the MCP server, treat that path as
authoritative rather than the platform default.

When a required profile is missing and modification of the local SSH-MCP
configuration is authorized:

1. Locate the active configuration using the explicit `--config` path when
   present; otherwise use the platform default above.
2. Read the existing configuration before modifying it.
3. Preserve existing defaults, profiles, policy, and formatting where
   practical.
4. Add only the required `[[profiles]]` entry, following existing profile
   conventions and using the least privilege appropriate for the target.
5. Never generate, replace, expose, or modify an existing SSH private key merely
   to create a profile. Reference the authorized key through `keyRef`.
6. Restart or reload the SSH-MCP server/client if required for configuration
   changes to become visible.
7. Verify the profile through `list-connections` before using it.

Do not silently fall back to direct SSH merely because an SSH-MCP profile is
missing. If the configuration cannot be modified because of filesystem,
client, tool, or authorization constraints, report the exact limitation.

On Windows, automatic discovery of `%APPDATA%\ssh-mcp\config.toml` has had
reported compatibility issues in some SSH-MCP versions. If the expected file
exists but SSH-MCP reports that no configuration was found, inspect how the MCP
server process is launched and prefer configuring an explicit
`--config <path>` rather than duplicating profiles or moving configuration
blindly.

## Preflight

1. Call `list-connections` before the first remote operation when the available
   profiles are not already known in the current conversation. If a required
   profile is absent, follow `Configuration And Profiles` rather than searching
   arbitrary filesystem locations or immediately falling back to direct SSH.
2. Select the returned profile by exact name. Pass `profile` explicitly when
   more than one profile exists or the intended target is not unquestionably
   the configured default.
3. Reuse established profile and connection information. Do not repeatedly
   rediscover it unless configuration or connection state may have changed.
4. Verify that the requested operation is authorized and operationally safe
   before choosing a tool. A tool being available is not authorization.

## Tool Selection

Prefer `read-command` whenever the authorized operation can be expressed as a
command SSH-MCP accepts as `read-only`.

Use `run-command` only when at least one of these is true:

* the authorized operation genuinely requires behavior that `read-command`
  cannot provide, including shell syntax or stateful interactive-session
  behavior;
* SSH-MCP classifies the required command above `read-only`.

Do not automatically retry a rejected `read-command` through `run-command`.
Inspect the classification or rejection reason first, then confirm the
operation remains authorized and safe before using the broader tool.

Use sessions only when state or duration requires them:

* `interactive`: several dependent commands need persistent CWD or environment;
* `background`: an authorized long-running command must outlive one tool call;
* one-shot commands: use `read-command` or `run-command`, not a session.

Use `sftp-download` only when the actual remote file is required locally. Use a
narrow read command when only metadata or a small excerpt is needed. Upload,
privileged, signal, and session-closing tools can alter the host and require the
corresponding explicit authorization.

## Command Construction

Keep `read-command` to one simple command. Shell-control syntax including pipes,
redirection, substitutions, variable expansion, grouping, separators, and
newlines disqualifies it from the `read-only` class even when the leading binary
is allowlisted.

Prefer native filtering and limiting options over shell pipelines. Examples:

* constrain paths and depth instead of recursively listing broad trees;
* request a bounded log time range or line count;
* select process columns rather than dumping full environments or command data;
* use `head`, `tail`, or a targeted `grep` directly when one command answers the
  question.

SSH-MCP applies configured command-length, timeout, and captured-output limits.
Do not depend on those limits as a querying strategy: narrow output at the
source, and treat truncated or timed-out diagnostics as incomplete evidence.

Do not place several independent diagnostics in one compound shell string.
Issue independent read calls concurrently when the client supports parallel
tool calls. Keep dependent commands sequential.

Avoid commands that may disclose credentials, private keys, tokens, full
environments, or application secrets. Output redaction is defense in depth, not
permission to request sensitive data.

## Classification And Policy

Classification describes command risk, not user intent. An allowlisted binary
can be reclassified when arguments execute or mutate, and a harmless binary not
on the allowlist can classify as `safe` rather than `read-only`.

Interpret failures before acting:

* invalid parameters such as an empty or over-length command: the request was
  rejected before policy and SSH execution. Correct the input.
* `Profile "..." not found`: profile resolution failed locally. Discover or fix
  the profile name; do not troubleshoot remote authentication yet.
* `only accepts read-only commands, got: ...`: classification mismatch. Simplify
  the command or deliberately choose an authorized broader tool.
* `POLICY_DENIED`: role, host tier, read-only profile, or denylist refused it.
  Do not retry alternate spellings or tools to bypass policy.
* `APPROVAL_DENIED`: the human declined. Stop unless the user later changes the
  authorization.
* `APPROVAL_UNAVAILABLE`: the client could not obtain required approval. Report
  the limitation; do not route around it.
* `QUOTA_EXCEEDED`: wait until the reported slot frees or use another profile
  only when that profile is independently the correct authorized target.
* SSH connection/authentication/host-key errors: transport or identity failure,
  not command-policy rejection. Check profile and connection state before retry.
* output ending in `[exit N]` or `[killed by SIG...]`: the remote command ran and
  failed. Use stderr and status to correct the command; do not call it an SSH
  failure.
* command timeout: the remote command may have been signalled, but heed any
  warning that it may still be running. Do not blindly rerun non-idempotent work.

Built-in never-allowed rules cannot be bypassed by approval or a broader role.
Never split, quote, wrap, or restate a command to evade classification, policy,
approval, or denylist controls.

## Sessions

Use unique names containing only letters, digits, dashes, or underscores.
Before opening a session, use `list-sessions` if collision or leaked-session
state is plausible; do not probe routinely when the name is known fresh.

Run only dependent stateful work in an interactive session. Specify its name in
subsequent `run-command` calls. Interactive sessions require a POSIX remote
shell and are not supported by Windows OpenSSH's default shell.

Poll background work with `read-session-output` using the smallest useful line
count. Close a session when its work is complete, recognizing that closing a
background session signals its remote process and is therefore not a read-only
cleanup operation.

## Efficiency Checklist

Before each call:

1. Is the exact profile already known?
2. Can one bounded `read-command` answer the question?
3. Can independent calls run concurrently without creating excess load?
4. Is the output constrained at the source?
5. If a previous call failed, am I responding to its specific failure class
   rather than blindly retrying or escalating?

Stop when the requested evidence is sufficient. Do not gather unrelated server
facts merely because an SSH connection is open.
