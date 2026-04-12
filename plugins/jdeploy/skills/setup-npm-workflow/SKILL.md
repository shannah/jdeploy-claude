---
description: Set up a GitHub Actions workflow for publishing a jDeploy app to npm with trusted publishing (OIDC provenance).
---

# jdeploy:setup-npm-workflow

Set up a GitHub Actions workflow for automated npm publishing of a jDeploy app.

## Overview

This skill creates a GitHub Actions workflow that publishes your jDeploy application to the npm registry whenever a GitHub Release is created or a version tag is pushed. It configures npm trusted publishing (OIDC provenance) so that published packages include verified provenance attestations.

Key features:
- **Automated publishing**: Triggered by version tags (`v*`) or snapshot branches
- **Trusted publishing**: Uses GitHub Actions OIDC tokens for npm provenance — no long-lived npm tokens required
- **Provenance attestation**: Published packages show verified build provenance on npmjs.com

## Instructions

### Step 1: Check Prerequisites

Verify the project is ready for npm workflow setup:

**Check for jDeploy configuration:**
```bash
node -p "JSON.stringify(require('./package.json').jdeploy, null, 2)" 2>/dev/null
```

If this fails or returns `undefined`, the project hasn't been set up with jDeploy yet. Tell the user to run `jdeploy:setup` first.

**Verify git repo with GitHub remote:**
```bash
git remote -v
```

Extract the GitHub owner and repo name from the remote URL. The remote must point to a GitHub repository.

**Check if workflow already exists:**
```bash
ls .github/workflows/jdeploy-npm.yml 2>/dev/null
```

If the file already exists, inform the user and ask if they want to overwrite it.

### Step 2: Ensure repository.url in package.json

npm trusted publishing (OIDC provenance) requires that `package.json` contains a `repository` field whose URL matches the GitHub repository. Without this, `npm publish` fails with:

```
E422 Unprocessable Entity - Error verifying sigstore provenance bundle:
Failed to validate repository information: package.json: "repository.url"
is "", expected to match "https://github.com/<owner>/<repo>" from provenance
```

**Detect the GitHub remote URL:**
```bash
git remote get-url origin
```

Parse the owner and repo from the remote URL (handles both HTTPS and SSH formats).

**Check current repository field:**
```bash
node -p "JSON.stringify(require('./package.json').repository)" 2>/dev/null
```

**Add or update the repository field** in `package.json`:

```json
"repository": {
  "type": "git",
  "url": "https://github.com/<owner>/<repo>"
}
```

The URL must use the `https://github.com/` format, not SSH (`git@github.com:`), to match what npm provenance verification expects.

### Step 3: Create the Workflow File

Create `.github/workflows/jdeploy-npm.yml`.

**Read the project's Java version from package.json:**
```bash
node -p "require('./package.json').jdeploy.javaVersion"
```

**Detect the build system:**
```bash
# Check for Maven
ls pom.xml 2>/dev/null && echo "Maven project detected"

# Check for Gradle
ls build.gradle build.gradle.kts 2>/dev/null && echo "Gradle project detected"
```

**Create the workflow directory:**
```bash
mkdir -p .github/workflows
```

**Write the workflow file** using this template:

```yaml
name: jDeploy npm Publish

on:
   push:
      branches:
         - '*-snapshot'
      tags:
         - 'v*'

concurrency:
   group: jdeploy-npm-${{ github.repository }}
   cancel-in-progress: false

jobs:
   publish:
      permissions:
         contents: read
         id-token: write
      runs-on: ubuntu-latest

      steps:
         - uses: actions/checkout@v3
         - name: Set up JDK
           uses: actions/setup-java@v3
           with:
              java-version: '{{ javaVersion from package.json }}'
              distribution: 'temurin'
         - name: Build with Maven
           run: mvn clean package -DskipTests
         - name: Publish to npm
           uses: shannah/jdeploy@master
           with:
              deploy_target: npm
              npm_token: ${{ secrets.NPM_TOKEN }}
```

**Adapt the build step to the project's build system:**

- **Maven**: `run: mvn clean package -DskipTests`
- **Gradle**: Replace the Maven build step with:
  ```yaml
  - name: Make gradlew executable
    run: chmod +x ./gradlew
  - name: Build with Gradle
    run: ./gradlew build -x test
  ```
- **Quarkus (Maven)**: `run: mvn clean package -DskipTests -Dquarkus.package.jar.type=uber-jar`
- **Quarkus (Gradle)**: `run: ./gradlew buildUberJar`

**Critical workflow settings:**

- **`id-token: write`** — Required for npm trusted publishing. GitHub Actions uses OIDC tokens to prove the package was built in CI. Without this permission, provenance signing fails and npm rejects the publish.
- **`contents: read`** — Sufficient because this workflow only publishes to npm, not to GitHub Releases. Unlike the GitHub releases workflow (`jdeploy.yml`), which needs `contents: write` to attach installer assets, the npm workflow only needs to read the repository.
- **`deploy_target: npm`** — Tells the jDeploy action to publish to npm instead of creating GitHub release assets.
- **`npm_token`** — Used as a fallback authentication method. With trusted publishing configured, the OIDC token is preferred, but the secret should still be set as a backup.

### Step 4: Guide npm Trusted Publishing Setup

Tell the user they need to configure npm to accept publishes from their GitHub Actions workflow. There are two options:

**Option A: npm Trusted Publishing (Recommended)**

npm trusted publishing uses OIDC so that GitHub Actions can publish without a long-lived access token. To set this up:

1. Go to [npmjs.com](https://www.npmjs.com/) and sign in
2. Navigate to the package settings page (or create the package first with an initial `npm publish` locally)
3. Under **"Publishing access"**, configure trusted publishing:
   - Set the **GitHub repository** (owner/repo)
   - Set the **workflow filename** to `jdeploy-npm.yml`
   - Set the **environment** to empty (default) unless you use GitHub environments

**Option B: npm Access Token**

If the user prefers token-based authentication:

1. Go to [npmjs.com](https://www.npmjs.com/) → **Access Tokens** → **Generate New Token**
2. Choose **Granular Access Token** or **Automation** token type
3. Grant publish permission for the package
4. Copy the token
5. Add it as a repository secret in GitHub:
   - Go to the repo → **Settings** → **Secrets and variables** → **Actions**
   - Click **New repository secret**
   - Name: `NPM_TOKEN`
   - Value: paste the token

### Step 5: Verify the Setup

Suggest the user test the workflow by creating a version tag:

```bash
# Ensure everything is committed and pushed
git add -A && git status
git push

# Create and push a version tag
git tag v$(node -p "require('./package.json').version")
git push origin v$(node -p "require('./package.json').version")
```

Then monitor the Actions tab:
- Go to `https://github.com/<owner>/<repo>/actions`
- Look for the "jDeploy npm Publish" workflow run
- Verify it completes successfully

After a successful run, verify the package on npm:
- Go to `https://www.npmjs.com/package/<package-name>`
- Check that the version matches and provenance badge is shown

---

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `E422 ... repository.url is ""` | Missing or empty `repository` field in `package.json` | Add `repository.url` matching the GitHub repo URL (see Step 2) |
| `E422 ... expected to match "https://github.com/..."` | `repository.url` doesn't match the GitHub repo | Update `repository.url` to `https://github.com/<owner>/<repo>` |
| `E404 Not Found - PUT` | npm token invalid or trusted publishing not configured | Verify `NPM_TOKEN` secret is set in GitHub repo settings, or configure trusted publishing on npmjs.com |
| `E403 Forbidden` | Package name already taken or insufficient permissions | Choose a different package name in `package.json`, or verify your npm account has publish access |
| `E401 Unauthorized` | No valid authentication method | Set up either trusted publishing (Option A) or an npm token (Option B) in Step 4 |
| Provenance signing fails | Missing `id-token: write` permission | Ensure the workflow has `permissions: { id-token: write }` on the job |
| Workflow not triggered | Tag doesn't match pattern | Ensure the tag starts with `v` (e.g., `v1.0.0`), or push to a branch ending in `-snapshot` |

---

## Quick Reference

```bash
# Check if workflow exists
ls .github/workflows/jdeploy-npm.yml

# Verify repository.url in package.json
node -p "require('./package.json').repository"

# Trigger a publish by creating a tag
git tag v1.0.0
git push origin v1.0.0

# Check package on npm after publishing
npm view <package-name>
```

## CLAUDE.md Reference

Consult these sections for detailed guidance:
- `package-json` — Full package.json schema and fields
- `github-workflows` — GitHub Actions workflow templates (including npm publishing)
- `build-validation` — Build and verification steps
