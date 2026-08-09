# Developer ID Signing and Notarization (Planned 2027)

Published releases are currently ad-hoc signed and not notarized. The project
will buy an Apple Developer Program membership and enable Developer ID signing,
notarization, and stapling in **2027**.

Until then, `.github/workflows/release.yml` accepts no signing secrets and
publishes the existing ad-hoc build with its SHA-256 checksum. It rejects a
partial secret set: signing is enabled only when all six secrets below exist.
This avoids a release that looks configured but is neither signed nor notarized.

The install script clears a stale quarantine attribute after checksum validation;
a browser-downloaded DMG needs the same `xattr` step documented in
[installation.md](installation.md).

## What to buy

An [Apple Developer Program](https://developer.apple.com/programs/) membership,
$99/year. An individual account is enough; TokenMyBar does not need an
organization.

## One-time setup for 2027

### 1. Create the Developer ID Application certificate

In Xcode: **Settings → Accounts → (your account) → Manage Certificates → + →
Developer ID Application**. Or generate a CSR and request it at
[developer.apple.com/account/resources/certificates](https://developer.apple.com/account/resources/certificates).

Then export it **with its private key** as a `.p12`:

1. Open **Keychain Access → login → My Certificates**.
2. Right-click the `Developer ID Application: … (TEAMID)` entry → **Export**.
3. Save as `.p12` and set a strong password — this becomes
   `DEVELOPER_ID_CERT_PASSWORD`.

Base64-encode it for GitHub (the secret must be a single line):

```bash
base64 -i DeveloperID.p12 | pbcopy
```

### 2. Create an app-specific password for notarytool

At [appleid.apple.com](https://appleid.apple.com) → **Sign-In and Security →
App-Specific Passwords**, generate one named e.g. `TokenMyBar notarytool`. This
is `AC_PASSWORD`. Your regular Apple ID password will not work.

Find your team ID at
[developer.apple.com/account](https://developer.apple.com/account) under
**Membership details**.

### 3. Add the repository secrets

**Settings → Secrets and variables → Actions → New repository secret**:

| Secret | Value |
|---|---|
| `DEVELOPER_ID_CERT_P12` | base64 of the `.p12` from step 1 |
| `DEVELOPER_ID_CERT_PASSWORD` | the `.p12` export password |
| `DEVELOPER_ID_APP` | the certificate's full common name, e.g. `Developer ID Application: Your Name (AB12CD34EF)` |
| `AC_APPLE_ID` | the Apple ID email that owns the membership |
| `AC_TEAM_ID` | the 10-character team ID, e.g. `AB12CD34EF` |
| `AC_PASSWORD` | the app-specific password from step 2 |

`DEVELOPER_ID_APP` must match the certificate name exactly. Confirm it with:

```bash
security find-identity -v -p codesigning
```

## Enabling and verifying the first signed release

After all six secrets are added, cut a release as usual (push a `vX.Y.Z` tag).
The workflow imports the certificate and its **Verify signature and
notarization** step runs the real checks — `spctl --assess` for Gatekeeper's
verdict and `stapler validate` for the notarization ticket — failing the release
if either does not pass.

Then confirm on a machine that has never run the app, ideally a fresh macOS user
account, that a **browser-downloaded** DMG (not `curl`, which sets no quarantine
flag) opens with no warning:

```bash
xattr -p com.apple.quarantine ~/Downloads/TokenMyBar-X.Y.Z.dmg  # should exist
spctl --assess --type open --context context:primary-signature -v ~/Downloads/TokenMyBar-X.Y.Z.dmg
```

## Renewal

Developer ID certificates last five years; the membership renews yearly. If the
membership lapses, notarization starts failing and the release workflow fails at
the verify step rather than publishing a silently unnotarized DMG.
