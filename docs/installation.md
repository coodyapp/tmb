# Installing TokenMyBar

TokenMyBar is a native macOS menu bar app. It requires **macOS 14 (Sonoma) or newer** on Apple Silicon.

> **Release integrity:** releases are ad-hoc signed, not notarized. Developer ID
> signing and notarization are planned for 2027. The install script verifies the
> published SHA-256 checksum and clears stale quarantine after installation.

## Option 1: Install script (recommended)

```bash
curl -fsSL https://raw.githubusercontent.com/coodyapp/token-my-bar/main/install.sh | bash
```

Downloads the latest release DMG, verifies its SHA-256 checksum, installs
**TokenMyBar.app** into `/Applications`, clears stale quarantine, and launches
it. Pass a version to pin one: `./install.sh 1.0.7`.

## Option 2: Homebrew

The cask lives in [coodyapp/homebrew-tap](https://github.com/coodyapp/homebrew-tap):

```bash
brew tap coodyapp/tap
brew install --cask token-my-bar
```

## Option 3: DMG (manual)

1. Download `TokenMyBar-<version>.dmg` from the [latest release](https://github.com/coodyapp/token-my-bar/releases/latest).
2. Open the DMG and drag **TokenMyBar.app** to `/Applications`.
3. Strip the quarantine flag the browser download added: `xattr -rd
   com.apple.quarantine /Applications/TokenMyBar.app`.
4. Launch TokenMyBar. The app lives in the menu bar only (no Dock icon).

## Option 4: Build from source

```bash
git clone https://github.com/coodyapp/token-my-bar.git
cd token-my-bar
apps/menubar/Scripts/package.sh
open apps/menubar/dist/TokenMyBar.app
```

See [development.md](development.md) for the full developer setup.

## First run

TokenMyBar reads usage from tools you already use — there is no separate sign-in:

- **Claude Code** — reads the OAuth credential Claude Code stored in the macOS
  Keychain. The first read triggers the standard macOS Keychain consent prompt;
  click **Allow** (or **Always Allow**).
- **OpenAI Codex** — reads `~/.codex/auth.json` written by the Codex CLI.
- **OpenCode** — imports the opencode.ai session cookie from a local browser: Arc,
  Chrome, Brave, Edge, Chromium, Vivaldi, or Firefox. For the Chromium-family
  browsers, decrypting the cookie also reads that browser's "Safe Storage"
  Keychain key, so expect a second Keychain prompt — but only for the browser that
  actually has an opencode.ai cookie. To skip browser import entirely, put the
  cookie in `~/.config/token-my-bar/config.toml` under `[opencode] cookie`
  (`chmod 600` it) — the `TOKEN_MY_BAR_OPENCODE_COOKIE` environment variable works
  for the CLI but never reaches the app, which Launch Services starts with no
  shell environment.
- **Google Antigravity** — while Antigravity runs, usage comes straight from
  the language server it hosts on your Mac (loopback only, no token needed).
  When it is closed — or its local server does not answer usably — TokenMyBar
  falls back to `~/.gemini/oauth_creds.json`,
  written by the Antigravity / Gemini sign-in. No Keychain prompt either way:
  the fallback is a plain file read. That stored token is short-lived and
  TokenMyBar never refreshes it, so if the vendor reports an expired sign-in,
  open Antigravity once and refresh.

Only Claude Code and the OpenCode browser import trigger Keychain prompts; Codex
and the Antigravity fallback are plain file reads, and Antigravity's primary
source never leaves the machine.

If a vendor shows **Sign in**, authenticate once in that vendor's own app and
refresh. See the [user guide](user-guide.md) for settings and troubleshooting,
and [privacy.md](privacy.md) for exactly what is read and stored.

## Uninstall

1. Quit TokenMyBar from the menu bar popover.
2. Delete `/Applications/TokenMyBar.app` (or `brew uninstall --cask token-my-bar`).
3. Optionally remove the snapshot cache: `rm -rf ~/Library/Application\ Support/token-my-bar`.
