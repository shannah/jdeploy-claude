---
description: Publish a jDeploy application to npm or GitHub. Creates native installers via GitHub Actions or publishes to npm registry.
---

# jdeploy:publish

Publish a jDeploy application to npm or GitHub.

## Overview

This skill publishes your application so users can install it. It supports two publish targets:

- **GitHub Releases**: Creates native installers (.exe, .dmg, .deb) via GitHub Actions
- **npm**: Publishes to the npm registry for `npm install -g` installation

The skill remembers your preferred publish target in `.jdeploy/config.json`.

## Instructions

### Step 1: Check for Saved Preference

Look for existing publish preference:

```bash
cat .jdeploy/config.json 2>/dev/null
```

If `.jdeploy/config.json` exists and has `publishTarget`, use that as the default.

### Step 2: Ask for Publish Target (if not saved)

If no preference is saved, ask the user:

**"Where would you like to publish?"**

1. **GitHub** (Recommended) - Creates native installers for Windows, macOS, and Linux. Users download from your releases page.

2. **npm** - Publishes to npm registry. Users install with `npm install -g <package-name>`.

### Step 3: Save Preference

Create `.jdeploy/config.json` to remember the choice:

```bash
mkdir -p .jdeploy
```

```json
{
  "publishTarget": "github"
}
```

Write this file so future publishes use the same target automatically.

---

## GitHub Publishing Flow

### Prerequisites Check

```bash
# Verify git repo
git status

# Verify remote exists
git remote -v

# Check for uncommitted changes
git status --porcelain
```

If there are uncommitted changes, ask the user to commit first.

### Check/Create GitHub Workflow

```bash
ls .github/workflows/jdeploy.yml 2>/dev/null
```

If the workflow doesn't exist, offer to create it. Use the template from CLAUDE.md section "github-workflows".

### Determine Version

Read version from `package.json`:

```bash
node -p "require('./package.json').version"
```

Ask user to confirm version or bump it (1.0.0 -> 1.0.1, 1.1.0, or 2.0.0).

If bumping, update `package.json`:

```bash
npm version patch  # or minor, or major
```

### Create Release

```bash
# Ensure changes are committed
git add -A && git commit -m "Release v<version>" || true

# Push to remote
git push

# Create the release
gh release create v<version> --title "v<version>" --notes "Release v<version>"
```

### Report Success

Tell the user:

1. **Release created**: Link to `https://github.com/<owner>/<repo>/releases/tag/v<version>`

2. **GitHub Actions building**: The jDeploy workflow is now building native installers

3. **Monitor progress**: Link to `https://github.com/<owner>/<repo>/actions`

4. **When complete**: Installers will be attached to the release (typically 5-10 minutes)

5. **Users can install from**: `https://github.com/<owner>/<repo>/releases/latest`

---

## npm Publishing Flow

### Prerequisites Check

```bash
# Verify npm login
npm whoami
```

If not logged in, instruct user:
```bash
npm login
```

### Verify Package Name

Check that `name` in `package.json` is a valid, available npm package name:
- Lowercase
- No spaces
- Can be scoped (@scope/name) or unscoped

### Build Project

**Maven:**
```bash
mvn clean package -DskipTests
```

**Gradle:**
```bash
./gradlew build -x test
```

### Publish to npm

```bash
npm publish
```

For scoped packages that should be public:
```bash
npm publish --access public
```

### Report Success

Tell the user:

1. **Published**: `https://www.npmjs.com/package/<package-name>`

2. **Users can install with**:
   ```bash
   npm install -g <package-name>
   ```

3. **Or run directly with**:
   ```bash
   npx <package-name>
   ```

---

## Changing Publish Target

If the user wants to switch targets:

```bash
# Edit .jdeploy/config.json
```

Change `publishTarget` to `"github"` or `"npm"`.

Or simply delete the file to be prompted again:
```bash
rm .jdeploy/config.json
```

---

## Troubleshooting

### GitHub Issues

**"gh: command not found"**:
Install GitHub CLI: https://cli.github.com/

**"authentication required"**:
```bash
gh auth login
```

**"release already exists"**:
Either delete the existing release or bump the version.

**"workflow not found"**:
Create `.github/workflows/jdeploy.yml` using the template.

### npm Issues

**"npm ERR! 403"**:
- Package name may be taken - choose a different name or use a scope
- You may not be logged in: `npm login`

**"npm ERR! 402 Payment Required"**:
- Scoped packages are private by default
- Use `npm publish --access public` for public packages

**"ENEEDAUTH"**:
```bash
npm login
```

---

## Quick Reference

```bash
# Publish to GitHub (after first time)
gh release create v1.0.0 --title "v1.0.0" --notes "Initial release"

# Publish to npm (after first time)
npm publish

# Check saved preference
cat .jdeploy/config.json

# Reset preference
rm .jdeploy/config.json
```

## Config File Location

Publish preferences are stored in the project at:
```
<project-root>/.jdeploy/config.json
```

Schema:
```json
{
  "publishTarget": "github"
}
```

Valid values for `publishTarget`: `"github"`, `"npm"`
