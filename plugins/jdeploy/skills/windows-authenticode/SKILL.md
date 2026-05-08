---
description: Configure Windows Authenticode code signing for an existing jDeploy project. Adds signing inputs to the GitHub Actions workflow for PFX/PKCS12 certificates or PKCS#11/HSM tokens (SafeNet, YubiKey, etc.) and guides setup of GitHub secrets.
---

# jdeploy:windows-authenticode

Configure Windows Authenticode code signing for an existing jDeploy project.

## Overview

Unsigned Windows `.exe` installers trigger SmartScreen warnings and may be flagged by antivirus software. Authenticode signing establishes publisher identity and reduces these warnings.

This skill wires Windows Authenticode signing into a project that is already set up with jDeploy. It will:

1. Detect the existing jDeploy GitHub Actions workflow (`.github/workflows/jdeploy.yml`)
2. Ask which signing method the user has available (local PFX/PKCS12 certificate or PKCS#11/HSM token)
3. Add the appropriate `win_signing_*` inputs to the `shannah/jdeploy` action step
4. Guide the user through encoding the certificate and storing it as a GitHub repository secret
5. Optionally configure environment variables for signing during local `jdeploy publish` runs

Signing uses the [jsign](https://ebourg.github.io/jsign/) library and is performed automatically during the packaging step. macOS and Linux bundles are unaffected.

## Instructions

### Step 1: Check Prerequisites

**Verify the project is set up with jDeploy:**

```bash
node -p "JSON.stringify(require('./package.json').jdeploy, null, 2)" 2>/dev/null
```

If this fails or returns `undefined`, tell the user to run `jdeploy:setup` first.

**Verify a GitHub Actions workflow exists:**

```bash
ls .github/workflows/jdeploy.yml 2>/dev/null
```

If the workflow does not exist, tell the user to run `jdeploy:publish` (or follow the GitHub workflow setup in `jdeploy:setup`) first so there is a workflow file to extend.

**Verify a GitHub remote is configured:**

```bash
git remote get-url origin
```

Parse the `<owner>/<repo>` from the URL — it is needed for the secrets-setup instructions in Step 4.

### Step 2: Ask Which Signing Method to Use

Ask the user which signing method they have:

1. **PFX / PKCS12 certificate file** — A `.pfx` or `.p12` file plus a password. Older CAs may still deliver these, but as of June 2023 the CA/Browser Forum requires all publicly trusted code-signing keys to be generated and stored in a hardware crypto module, so new OV/EV certs are generally HSM-backed (use option 2 or 3 instead).

2. **DigiCert KeyLocker (cloud HSM)** (Recommended for new certs) — DigiCert's cloud HSM signing service. Works on standard GitHub-hosted runners (`ubuntu-latest`) — no self-hosted runner required. This is the most common path for new OV/EV code-signing certs purchased from DigiCert.

3. **On-premise PKCS#11 / Hardware Security Module** — A physical token (SafeNet eToken, YubiKey, etc.) or an on-prem HSM. Requires a self-hosted GitHub runner with the HSM driver installed.

4. **No certificate yet** — If the user has not obtained a certificate, briefly describe the options and stop. Do not generate a self-signed certificate for production use; SmartScreen will still warn on self-signed binaries because the root is not trusted. Self-signed certs are only useful for internal/test distribution. Point them to a code-signing CA, then resume this skill once they have the cert in hand.

### Step 3A: PFX Signing Setup

#### 3A.1. Prepare the certificate

Ask the user for the absolute path to their `.pfx` (or `.p12`) file and the keystore password. **Do not** ask them to paste either into chat — explain that the password will be stored as a GitHub secret and that the user will run a local command to encode the cert.

Show the user the command to base64-encode the certificate (do **not** run this for them — the cert may be sensitive and should be handled on their machine):

```bash
# macOS
base64 -i certificate.pfx | tr -d '\n' | pbcopy

# Linux
base64 -w 0 certificate.pfx | xclip -selection clipboard

# Windows (PowerShell)
[Convert]::ToBase64String([IO.File]::ReadAllBytes("certificate.pfx")) | Set-Clipboard
```

#### 3A.2. Add GitHub repository secrets

Show the user the `gh` commands to set the secrets (substitute `<owner>/<repo>` from Step 1):

```bash
# Paste the base64 text from the clipboard when prompted
gh secret set WIN_SIGNING_CERT_BASE64 --repo <owner>/<repo>

# Set the keystore password
gh secret set WIN_SIGNING_PASSWORD --repo <owner>/<repo>
```

Alternatively, they can add the secrets via the GitHub UI: **Settings → Secrets and variables → Actions → New repository secret**.

#### 3A.3. Update the GitHub Actions workflow

Read the current workflow:

```bash
cat .github/workflows/jdeploy.yml
```

Locate the `shannah/jdeploy@...` action step (typically named "Build App Installer Bundles" or similar) and add the `win_signing_*` inputs under its `with:` block. The minimal additions are:

```yaml
      - name: Build App Installer Bundles
        uses: shannah/jdeploy@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          # ...existing inputs...
          win_signing_certificate: ${{ secrets.WIN_SIGNING_CERT_BASE64 }}
          win_signing_password: ${{ secrets.WIN_SIGNING_PASSWORD }}
```

Optional inputs (ask the user whether to include them):

| Input | Purpose |
|---|---|
| `win_signing_description` | Human-readable description embedded in the signature (e.g. the app's title) |
| `win_signing_url` | URL embedded in the signature (e.g. the app's website or repo URL) |
| `win_signing_key_alias` | Alias of the signing key inside the keystore (defaults to the first alias) |
| `win_signing_key_password` | Private key password if it differs from the keystore password (rare for `.pfx`) |
| `win_signing_timestamp_url` | RFC 3161 timestamp server (defaults to `http://timestamp.digicert.com`) |
| `win_signing_hash_algorithm` | Hash algorithm (defaults to `SHA-256`) |

Use the `Edit` tool to insert the new inputs into the existing `with:` block — do not overwrite the whole workflow. Preserve indentation (workflow files are typically indented with 3 or 6 spaces — match the surrounding lines).

Skip to **Step 4**.

### Step 3B: DigiCert KeyLocker Signing Setup

DigiCert KeyLocker is a cloud HSM. The signing key never leaves DigiCert's HSM, but signing is invoked via DigiCert's `smtools` CLI and PKCS#11 provider library that gets installed on the runner at job time. This works on the standard `ubuntu-latest` GitHub-hosted runner — no self-hosted infrastructure required.

#### 3B.1. Gather DigiCert credentials

The user needs the following from their DigiCert ONE / KeyLocker account:

- **API Key** (from DigiCert ONE → KeyLocker → API Tokens)
- **Client Authentication Certificate** — a `.p12` file downloaded from KeyLocker, plus its password
- **Host URL** — the SM_HOST URL for their DigiCert account (e.g. `https://clientauth.one.digicert.com`)
- **Key alias** — the alias of the signing keypair as registered in KeyLocker (visible via `smctl keypair ls` after setup)

#### 3B.2. Add GitHub repository secrets and variables

```bash
# Secrets
gh secret set SM_API_KEY --repo <owner>/<repo>
gh secret set SM_CLIENT_CERT_PASSWORD --repo <owner>/<repo>

# Base64-encode the client cert .p12 (run on the user's machine):
#   macOS:   base64 -i ClientCert.p12 | tr -d '\n' | pbcopy
#   Linux:   base64 -w 0 ClientCert.p12 | xclip -selection clipboard
gh secret set SM_CLIENT_CERT_FILE_B64 --repo <owner>/<repo>

# Variable (not a secret — the host URL is not sensitive)
gh variable set SM_HOST --repo <owner>/<repo> --body "https://clientauth.one.digicert.com"
```

#### 3B.3. Update the GitHub Actions workflow

The workflow needs three new steps **before** the `shannah/jdeploy` step: install `smtools`, write the client cert and PKCS#11 config, and sync KeyLocker certificates. Then the `shannah/jdeploy` step is configured for PKCS#11 signing and given the DigiCert env vars.

Insert these steps before the existing "Build App Installer Bundles" step in `.github/workflows/jdeploy.yml`:

```yaml
      - name: Install DigiCert smtools
        env:
          SM_API_KEY: ${{ secrets.SM_API_KEY }}
        run: |
          if [ -z "$SM_API_KEY" ]; then
            echo "SM_API_KEY not configured, skipping smtools install"
            exit 0
          fi
          curl -L --fail --show-error \
            -o smtools.tar.gz \
            https://pki-downloads.digicert.com/stm/latest/smtools-linux-x64.tar.gz
          tar -xzf smtools.tar.gz
          sudo mkdir -p /opt/digicert
          sudo cp -r smtools-linux-x64/* /opt/digicert/
          sudo chmod -R +x /opt/digicert/
          echo "/opt/digicert" >> $GITHUB_PATH
          echo "DigiCert smtools installed to /opt/digicert"

      - name: Configure DigiCert KeyLocker
        env:
          SM_API_KEY: ${{ secrets.SM_API_KEY }}
        run: |
          if [ -z "$SM_API_KEY" ]; then
            echo "SM_API_KEY not configured, skipping DigiCert configuration"
            exit 0
          fi
          echo "${{ secrets.SM_CLIENT_CERT_FILE_B64 }}" | base64 -d > /tmp/digicert-client-cert.p12
          cat > /tmp/digicert-pkcs11.cfg <<'CFGEOF'
          name = DigiCertKeyLocker
          library = /opt/digicert/smpkcs11.so
          slotListIndex = 0
          CFGEOF
          echo "DigiCert KeyLocker PKCS#11 configured"

      - name: Sync DigiCert KeyLocker certificates
        env:
          SM_API_KEY: ${{ secrets.SM_API_KEY }}
          SM_CLIENT_CERT_FILE: /tmp/digicert-client-cert.p12
          SM_CLIENT_CERT_PASSWORD: ${{ secrets.SM_CLIENT_CERT_PASSWORD }}
          SM_HOST: ${{ vars.SM_HOST }}
        run: |
          if [ -z "$SM_API_KEY" ]; then
            echo "SM_API_KEY not configured, skipping certificate sync"
            exit 0
          fi
          smctl keypair ls
          echo "Certificate sync complete"
```

Then update the existing `shannah/jdeploy` step to enable PKCS#11 signing and pass the DigiCert env vars:

```yaml
      - name: Build App Installer Bundles
        uses: shannah/jdeploy@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          # ...existing inputs...
          win_signing_keystore_type: PKCS11
          win_signing_pkcs11_config: /tmp/digicert-pkcs11.cfg
          win_signing_password: ${{ secrets.SM_API_KEY }}
        env:
          SM_API_KEY: ${{ secrets.SM_API_KEY }}
          SM_CLIENT_CERT_FILE: /tmp/digicert-client-cert.p12
          SM_CLIENT_CERT_PASSWORD: ${{ secrets.SM_CLIENT_CERT_PASSWORD }}
          SM_HOST: ${{ vars.SM_HOST }}
```

**Key points specific to KeyLocker:**

- `win_signing_password` is set to the **API key**, not a token PIN. The PKCS#11 provider uses the API key as the login password and then authenticates to KeyLocker via the client cert + `SM_*` environment variables.
- The `SM_*` env vars must be on the `shannah/jdeploy` step itself — the PKCS#11 library reads them at signing time.
- If the keypair has more than one alias, add `win_signing_key_alias: <alias>` matching the alias shown by `smctl keypair ls`. With a single keypair the first alias is used automatically.
- Each step that uses smtools or jsign-PKCS#11 short-circuits when `SM_API_KEY` is empty. This lets contributors who don't have access to KeyLocker secrets run the workflow on forks without errors — they'll get unsigned `.exe` bundles instead.

Skip to **Step 4**.

### Step 3C: On-premise PKCS#11 / HSM Signing Setup

PKCS#11 signing requires the HSM's PKCS#11 provider library on the machine that runs the signing step. GitHub-hosted runners do **not** have these drivers, so this path requires a **self-hosted runner**.

Tell the user this up front. If they don't have a self-hosted runner with the HSM driver installed, they should either:

- Set one up (see [GitHub's self-hosted runner docs](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners)), or
- Switch to a PFX-backed certificate (Step 3A) if their CA offers one.

Once the runner is ready:

#### 3B.1. Prepare the PKCS#11 config file on the runner

The runner needs a PKCS#11 provider config file. Example for SafeNet:

```
name = SafeNet
library = /opt/safenet/lib/libCryptoki2_64.so
```

Place this file at a known path on the runner (e.g. `/opt/safenet/pkcs11.cfg`).

#### 3B.2. Add GitHub repository secrets

Only the token PIN needs to be a secret:

```bash
gh secret set HSM_TOKEN_PIN --repo <owner>/<repo>
```

#### 3B.3. Update the GitHub Actions workflow

The job must run on the self-hosted runner (`runs-on: self-hosted` instead of `ubuntu-latest`). Edit `.github/workflows/jdeploy.yml`:

1. Change the job's `runs-on:` from `ubuntu-latest` to `self-hosted` (or the appropriate runner label).
2. Add the `win_signing_*` inputs to the `shannah/jdeploy` step:

```yaml
      - name: Build App Installer Bundles
        uses: shannah/jdeploy@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          # ...existing inputs...
          win_signing_keystore_type: PKCS11
          win_signing_pkcs11_config: /opt/safenet/pkcs11.cfg
          win_signing_password: ${{ secrets.HSM_TOKEN_PIN }}
          win_signing_key_alias: <alias-of-signing-key-in-token>
```

Ask the user for the actual `pkcs11.cfg` path on their runner and the alias of the signing key in the token, and substitute them in.

### Step 4: Optional — Configure Local Signing for `jdeploy publish`

If the user runs `jdeploy publish` locally (rather than only via GitHub Actions), they can configure signing via environment variables. Offer this as an optional step.

For PFX:

```bash
export JDEPLOY_WIN_KEYSTORE_PATH="/absolute/path/to/certificate.pfx"
export JDEPLOY_WIN_KEYSTORE_PASSWORD="..."
# Optional:
export JDEPLOY_WIN_SIGN_DESCRIPTION="My App"
export JDEPLOY_WIN_SIGN_URL="https://example.com"
```

For PKCS#11:

```bash
export JDEPLOY_WIN_KEYSTORE_TYPE=PKCS11
export JDEPLOY_WIN_PKCS11_CONFIG="/absolute/path/to/pkcs11.cfg"
export JDEPLOY_WIN_KEYSTORE_PASSWORD="<token-pin>"
export JDEPLOY_WIN_KEY_ALIAS="<key-alias>"
```

Recommend the user put these in a `.env` file (added to `.gitignore`) or their shell profile — **never** commit the password.

If neither `JDEPLOY_WIN_KEYSTORE_PATH` (PFX) nor `JDEPLOY_WIN_PKCS11_CONFIG` + `JDEPLOY_WIN_KEYSTORE_TYPE=PKCS11` (HSM) are set, jDeploy silently skips signing and produces unsigned `.exe` bundles. This is the desired behavior for contributors who don't have access to the cert.

### Step 5: Verify the Configuration

Suggest the user trigger a release to verify signing works end-to-end:

```bash
# Bump and tag a new version
npm version patch
git push && git push --tags

# Or create a release via gh
gh release create v$(node -p "require('./package.json').version") \
  --title "v$(node -p "require('./package.json').version")" \
  --notes "Test release with Authenticode signing"
```

Monitor the workflow in the Actions tab. The signing step happens during the "Build App Installer Bundles" step — look for log lines mentioning `jsign` or "Signing".

After the workflow completes, download the Windows `.exe` from the release and verify the signature. On Windows:

```powershell
# PowerShell
$sig = Get-AuthenticodeSignature "MyApp-Installer.exe"
Write-Host "Status: $($sig.Status)"
Write-Host "Signer: $($sig.SignerCertificate.Subject)"
Write-Host "Timestamp: $($sig.TimeStamperCertificate.Subject)"
```

Or right-click the `.exe` → **Properties** → **Digital Signatures** tab.

A valid signature shows:
- `Status: Valid` (or `UnknownError` for self-signed certs without a trusted root — expected)
- `Signer:` matches the certificate subject
- A timestamp from the timestamp server (DigiCert by default)

---

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| Workflow runs but installers are unsigned | Required secrets not set, or input names misspelled | Verify `WIN_SIGNING_CERT_BASE64` and `WIN_SIGNING_PASSWORD` exist in repo secrets and are referenced correctly in the workflow |
| `KeystoreException: Failed to load the keystore` | Wrong password, or base64 of cert was corrupted (e.g. line breaks) | Re-encode the PFX with `base64 -w 0` (Linux) or `tr -d '\n'` (macOS) to strip newlines, then update the secret |
| `Unable to find a key alias` | Certificate has no alias matching `win_signing_key_alias` | Omit `win_signing_key_alias` to use the first alias automatically, or run `keytool -list -keystore cert.pfx -storetype PKCS12` to find the actual alias |
| `SignerCertificate is null` after signing | Older jsign version with OpenSSL 3.x certs | Use a current jDeploy release (jsign 7.0+) |
| `0x80096019` basic constraint error | Self-signed cert was generated without proper extensions | Real CA-issued certs don't have this; for test certs include `basicConstraints=CA:FALSE`, `keyUsage=digitalSignature`, `extendedKeyUsage=codeSigning` |
| PKCS#11 step fails with `CKR_PIN_INCORRECT` | Wrong PIN in `HSM_TOKEN_PIN` secret (or wrong API key for KeyLocker) | Reset the secret with the correct value |
| PKCS#11 fails with `library not found` | PKCS#11 driver not installed on the runner | For on-prem HSM: install the vendor's PKCS#11 library on the self-hosted runner. For DigiCert KeyLocker: ensure the "Install DigiCert smtools" step ran successfully |
| KeyLocker `smctl keypair ls` returns 401/403 | `SM_API_KEY` invalid, or client cert (`SM_CLIENT_CERT_FILE_B64`) doesn't match the API key's account | Regenerate the API key and re-download the client cert from DigiCert ONE; update both secrets |
| KeyLocker signing fails with `cert chain not found` | The keypair in KeyLocker has no associated certificate, or the cert hasn't finished issuance | In DigiCert ONE, verify the cert is "Issued" and bound to the keypair; re-run `smctl keypair ls` to confirm |
| KeyLocker signing fails with `connection refused` to SM_HOST | Wrong `SM_HOST` for the user's DigiCert account region | Check the URL in DigiCert ONE → Account Settings; update the `SM_HOST` repository variable |
| SmartScreen still warns on the signed binary | OV certs build reputation over time; new certs trigger SmartScreen until enough downloads occur | Use an EV certificate for instant SmartScreen reputation, or wait/build reputation organically |

---

## Quick Reference

```bash
# Check what's configured
grep -A 1 "win_signing" .github/workflows/jdeploy.yml
gh secret list --repo <owner>/<repo>

# Re-encode a PFX
base64 -w 0 certificate.pfx > cert.b64

# Set/rotate the cert secret
gh secret set WIN_SIGNING_CERT_BASE64 --repo <owner>/<repo> < cert.b64

# Trigger a build
gh release create v1.0.1 --title "v1.0.1" --notes "..."
```

## Reference: All `win_signing_*` Action Inputs

| Input | Maps To | Description |
|---|---|---|
| `win_signing_certificate` | `JDEPLOY_WIN_KEYSTORE_PATH` | Base64-encoded PFX/PKCS12 certificate; auto-decoded to a temp file |
| `win_signing_password` | `JDEPLOY_WIN_KEYSTORE_PASSWORD` | Keystore password or PKCS#11 token PIN |
| `win_signing_keystore_type` | `JDEPLOY_WIN_KEYSTORE_TYPE` | `PKCS12` (default), `JKS`, or `PKCS11` |
| `win_signing_pkcs11_config` | `JDEPLOY_WIN_PKCS11_CONFIG` | Path to PKCS#11 provider config on the runner |
| `win_signing_key_alias` | `JDEPLOY_WIN_KEY_ALIAS` | Alias of the signing key (defaults to first alias) |
| `win_signing_key_password` | `JDEPLOY_WIN_KEY_PASSWORD` | Private key password if different from keystore password |
| `win_signing_timestamp_url` | `JDEPLOY_WIN_TIMESTAMP_URL` | RFC 3161 timestamp server (default `http://timestamp.digicert.com`) |
| `win_signing_hash_algorithm` | `JDEPLOY_WIN_HASH_ALGORITHM` | Default `SHA-256` |
| `win_signing_description` | `JDEPLOY_WIN_SIGN_DESCRIPTION` | Description embedded in the signature |
| `win_signing_url` | `JDEPLOY_WIN_SIGN_URL` | URL embedded in the signature |

## Security Notes

- **Never** commit `.pfx`/`.p12` files or passwords to the repo. Add `*.pfx` and `*.p12` to `.gitignore` if certs live anywhere in the project tree.
- The jDeploy action writes the decoded PFX to `${{ runner.temp }}` and cleans it up after the workflow run (including on failure).
- Rotate the keystore password and re-issue the cert if a secret is ever exposed in logs or accidentally committed.
- For production releases, prefer EV certificates — they get immediate SmartScreen reputation and require HSM-backed keys, which makes accidental key exfiltration much harder.
