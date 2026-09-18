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

## Codex Desktop

Codex Desktop may be installed without a `codex` command on `PATH`. The v0.2.0
setup command checks only for that command, so it can reject a working desktop
installation. Link the packaged skills manually from `cmd.exe`:

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

Restart Codex before using `/omm-scan`. On v0.2.0, `omm setup --list` may still
incorrectly show Codex as not installed because this workaround installs the
skills without changing the package's detection code.

The original Windows report is tracked in
[#24](https://github.com/oh-my-mermaid/oh-my-mermaid/issues/24). Native Codex
Desktop detection and Windows directory junction support are tracked in
[#33](https://github.com/oh-my-mermaid/oh-my-mermaid/pull/33).
