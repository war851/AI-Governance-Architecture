# Windows Environment Rules

## Overview

This project runs on Windows 10 with PowerShell. All shell commands and scripts must be Windows-compatible.

## Requirements (MUST)

- Use PowerShell syntax for all shell operations unless explicitly using Git Bash
- Use forward slashes in configuration files and glob patterns
- Use `2>$null` instead of `2>/dev/null` for error suppression
- Use semicolons or pipeline operators instead of `&&` for command chaining in PowerShell
- Test file paths with spaces by quoting them: `"path with spaces"`

## Conventions (SHOULD)

- Prefer cross-platform commands when both Unix and Windows variants exist
- Use `Get-ChildItem` instead of `ls -la` (or use `dir`)
- Use `Remove-Item` instead of `rm -rf` (or use `del /s`)
- Use `$env:VARIABLE` instead of `$VARIABLE` for environment variable access in PowerShell
- Use `Test-Path` instead of `[ -f file ]` for file existence checks

## Anti-Patterns (NEVER)

- NEVER assume bash is the default shell — PowerShell is
- NEVER use Unix-only utilities without checking for Windows alternatives
- NEVER use symlinks without checking if developer mode is enabled
- NEVER hardcode Unix path separators in runtime code that resolves file paths
