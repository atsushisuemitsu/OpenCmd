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
- Commands in `.reg` are chained using `^&` (escaped `&` for cmd.exe) rather than `&&` to ensure reliable execution through the PowerShell → cmd.exe pipeline
- The `%V` shell variable in commands expands to the target folder path at runtime
- `pushd` is used instead of `cd /d` because it supports UNC paths (`\\server\share`)
- `HasLUAShield` value displays the UAC shield icon on the menu item
- Windows 11 modern context menu (top-level) requires COM `IExplorerCommand` + MSIX packaging — registry-based entries only appear under "Show more options"

## Testing

No automated tests. Manual verification:
1. Run `install.reg` as administrator
2. Right-click folder background/icon/drive → "その他のオプションを確認" → verify menu item appears
3. Click menu item → UAC prompt → verify CMD opens at correct path
4. Run `uninstall.reg` → verify menu item is removed
