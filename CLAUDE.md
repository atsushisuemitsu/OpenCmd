# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenCmd is a Windows registry-based tool that adds "Open CMD as Administrator" to the Windows right-click context menu. It consists of `.reg` files only — no build system, no compiled code.

## Architecture

- `install.reg` — Adds registry entries under `HKEY_CLASSES_ROOT\Directory\Background\shell\OpenCmdAsAdmin`, `HKEY_CLASSES_ROOT\Directory\shell\OpenCmdAsAdmin`, and `HKEY_CLASSES_ROOT\Drive\shell\OpenCmdAsAdmin`
- `uninstall.reg` — Removes those same registry keys (using `-` prefix syntax)
- Elevation is handled via `PowerShell -windowstyle hidden -Command "Start-Process cmd.exe ... -Verb RunAs"` rather than the `runas` special key name, to avoid conflicts with existing registry entries

## Key Technical Details

- `.reg` files use UTF-16 LE with BOM encoding (required for Japanese characters to display correctly in the registry editor and context menus). When editing `.reg` files, always re-encode to UTF-16 LE BOM before committing.
- Commands in `.reg` are chained using `&&` to ensure reliable execution through the PowerShell → cmd.exe pipeline
- The `%V` shell variable in commands expands to the target folder path at runtime
- `pushd` is used instead of `cd /d` because it supports UNC paths (`\\server\share`)
- `HasLUAShield` value displays the UAC shield icon on the menu item
- Windows 11 modern context menu (top-level) requires COM `IExplorerCommand` + MSIX packaging — registry-based entries only appear under "Show more options"

## Editing .reg Files — CRITICAL

**NEVER use the Write tool to edit .reg files directly.** The Write tool outputs UTF-8, but `.reg` files with Japanese text require UTF-16 LE with BOM + CRLF line endings. Instead:

1. Write a PowerShell script (`_gen.ps1`) using the Write tool
2. Build Japanese text from Unicode code points: `[char]0x7BA1,[char]0x7406,...` (avoids PS 5.1 cp932 encoding issues)
3. Build line endings explicitly: `$r = [char]13 + [char]10` (CRLF)
4. Build quotes/backslashes as char variables to avoid escaping confusion
5. Add UTF-8 BOM to the `.ps1` file so PowerShell 5.1 reads it correctly
6. Write output with `[System.IO.File]::WriteAllText($path, $content, [System.Text.UnicodeEncoding]::new($false, $true))`
7. Delete the temporary `.ps1` file after execution

## Testing

No automated tests. Manual verification:
1. Run `install.reg` as administrator
2. Right-click folder background/icon/drive → "その他のオプションを確認" → verify menu item appears
3. Click menu item → UAC prompt → verify CMD opens at correct path
4. Run `uninstall.reg` → verify menu item is removed
