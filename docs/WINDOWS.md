# Windows setup errors for installed tools

Use this workaround when an installed tool is not detected on Windows and
`omm setup` reports either of these errors:

```text
error: Claude Code is not installed on this machine.
error: Codex is not installed on this machine.
```

The issue affects the published npm v0.2.0 package, which predates the
cross-platform command detection fix in
[#22](https://github.com/oh-my-mermaid/oh-my-mermaid/pull/22).

Check the published and installed versions from `cmd.exe`:

```cmd
npm view oh-my-mermaid version
omm --version
```

If a version newer than `0.2.0` is available, update and retry normal setup
before using this workaround:

```cmd
npm install -g oh-my-mermaid@latest
omm setup
```

## PowerShell script execution policy

If PowerShell reports that `omm.ps1` cannot be loaded because running scripts
is disabled, use the npm-generated Command Prompt shim instead:

```powershell
omm.cmd --version
omm.cmd setup
```

Use `omm.cmd` in place of `omm` for other CLI commands in that PowerShell
session. This workaround does not require changing the system execution policy.

## Claude Code

Confirm that the Claude Code CLI is available:

```cmd
where claude
claude --version
```

Then run the same plugin commands that `omm setup claude` would run after
detection succeeds:

```cmd
claude plugin marketplace add oh-my-mermaid/oh-my-mermaid
claude plugin install oh-my-mermaid
```

## Codex

A working Codex installation may not expose a `codex` command on `PATH`. The
v0.2.0 setup command checks only for that command, so it can reject Codex even
when it is available through the desktop app or CLI. Link the packaged skills
manually from `cmd.exe`:

```cmd
for /f "delims=" %I in ('npm root -g') do set "OMM_NPM_ROOT=%I"
set "OMM_SOURCE=%OMM_NPM_ROOT%\oh-my-mermaid\skills"
set "OMM_TARGET=%USERPROFILE%\.agents\skills\oh-my-mermaid"
if not exist "%USERPROFILE%\.agents\skills" mkdir "%USERPROFILE%\.agents\skills"
mklink /J "%OMM_TARGET%" "%OMM_SOURCE%"
```

If `mklink` reports that the target already exists, inspect it instead of
deleting or replacing it automatically:

```cmd
dir /AL "%USERPROFILE%\.agents\skills"
dir "%OMM_TARGET%"
```

Verify the three installed skills:

```cmd
dir "%OMM_TARGET%\omm-scan\SKILL.md"
dir "%OMM_TARGET%\omm-push\SKILL.md"
dir "%OMM_TARGET%\omm-view\SKILL.md"
```

Restart Codex so it reloads the skills. Then enter the following as a normal
Codex prompt:

```text
omm scan

When running oh-my-mermaid from Windows PowerShell, use `omm.cmd` instead of
`omm`. Do not change the system execution policy.

Before piping non-ASCII content through stdin from Windows PowerShell 5.1,
configure UTF-8:

$OutputEncoding = [Console]::OutputEncoding =
  [System.Text.UTF8Encoding]::new()
```

`/omm-scan` is the Claude Code slash command, not a Codex slash command. On
v0.2.0, `omm setup --list` may still incorrectly show Codex as not installed
because this workaround installs the skills without changing the package's
detection code.

The original Windows report is tracked in
[#24](https://github.com/oh-my-mermaid/oh-my-mermaid/issues/24). Native Codex
Desktop detection and Windows directory junction support are tracked in
[#33](https://github.com/oh-my-mermaid/oh-my-mermaid/pull/33).
