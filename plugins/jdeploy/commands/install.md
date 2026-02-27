# jdeploy:install

Install a jDeploy application locally for testing.

## Overview

This skill installs the application on the local machine, creating:
- Desktop shortcuts/Start menu entries
- CLI commands in PATH (if configured)
- File associations (if configured)

This lets you test the full installation experience before publishing.

## Instructions

### Step 1: Verify Project Setup

Check that the project has jDeploy configured:

```bash
ls package.json 2>/dev/null || echo "No package.json found"
```

If no `package.json` exists, suggest running `/jdeploy:setup` first.

### Step 2: Build the Project

Ensure the project is built with the latest changes:

**Maven:**
```bash
mvn clean package -DskipTests
```

**Gradle:**
```bash
./gradlew build -x test
```

### Step 3: Install Locally

```bash
npx jdeploy install
```

This will:
1. Package the application
2. Install it to the user's local application directory
3. Create desktop shortcuts (GUI apps)
4. Add CLI commands to PATH (if `jdeploy.commands` configured)
5. Register file associations (if configured)

### Step 4: Verify Installation

**For GUI apps:**
- Look for the app in your Applications folder (macOS) or Start menu (Windows)
- Try launching it from the shortcut

**For CLI apps:**
- Open a new terminal (to pick up PATH changes)
- Run the command name specified in `package.json`:
  ```bash
  your-command-name --help
  ```

**Check installed location:**
- macOS: `~/Applications/` or `/Applications/`
- Windows: `%LOCALAPPDATA%\Programs\`
- Linux: `~/.local/share/applications/`

## Uninstalling

To remove a locally installed app:

```bash
npx jdeploy uninstall
```

Or manually delete from the installation directory.

## Options

| Option | Description |
|--------|-------------|
| `--global` | Install system-wide (may require sudo) |

## Troubleshooting

**"Command not found" after install:**
- Open a new terminal window (PATH changes require new session)
- Check that `jdeploy.commands` is configured in `package.json`

**"Permission denied":**
- Try without `--global` for user-local install
- Or use `sudo npx jdeploy install --global`

**App doesn't appear in Applications:**
- Check the `jdeploy.title` in `package.json` - that's the display name
- Try logging out and back in to refresh desktop entries

**Icon not showing:**
- Ensure `icon.png` exists in project root
- Icon must be PNG format, preferably 256x256 or larger

## Quick Reference

```bash
# Build and install (Maven)
mvn clean package && npx jdeploy install

# Build and install (Gradle)
./gradlew build && npx jdeploy install

# Uninstall
npx jdeploy uninstall

# Reinstall (uninstall + install)
npx jdeploy uninstall && npx jdeploy install
```

## What Gets Installed

| App Type | What's Created |
|----------|---------------|
| GUI app | Desktop shortcut, Start menu entry, optional file associations |
| CLI app | Command in PATH |
| Service | Command in PATH with `service` subcommands |
| MCP Server | Command in PATH, optional AI tool registration |
