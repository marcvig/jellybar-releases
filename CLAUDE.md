# JellyBar Releases

Public release repo for [JellyBar](https://jellybar.app). Source code lives in the private `jellybar` repo.

## What lives here

- `appcast.xml` — Sparkle update feed, served via GitHub Pages at `https://marcvig.github.io/jellybar-releases/appcast.xml`
- GitHub Releases — notarized DMG assets (e.g. `JellyBar-0.2.0.dmg`)
- `README.md` — public-facing install instructions

## Per-release steps

Full runbook: `docs/RELEASE-CHECKLIST.md` in the private `jellybar` source repo.

Short version:
1. Build + notarize in the source repo: `Scripts/make-dmg.sh --notarize`
2. `make-dmg.sh` prints an `<enclosure>` block — paste it at the TOP of `appcast.xml` as a new `<item>` (newest first)
3. Commit + push `appcast.xml` → GitHub Pages auto-deploys (~60 s); verify: `curl -s https://marcvig.github.io/jellybar-releases/appcast.xml | grep shortVersionString | head -1`
4. Create GitHub Release: `gh release create v<VERSION> --title "JellyBar <VERSION>" --notes "See https://jellybar.app" /tmp/JellyBar-<VERSION>.dmg`
5. Update `sha256` + `version` in `~/Documents/GitHub/homebrew-tap/Casks/jellybar.rb`, then push

## Naming conventions

- DMG: `JellyBar-<VERSION>.dmg` (e.g. `JellyBar-0.2.0.dmg`)
- Release tag: `v<VERSION>` (e.g. `v0.2.0`)
- Appcast `<sparkle:version>`: CFBundleVersion integer (e.g. `2`)
- Appcast `<sparkle:shortVersionString>`: CFBundleShortVersionString (e.g. `0.2.0`)

## Never move the appcast URL

`https://marcvig.github.io/jellybar-releases/appcast.xml` is baked into every shipped binary as `SUFeedURL`.
Moving it breaks Sparkle updates for all existing installs. Enclosure URLs inside the appcast can change freely.

## EdDSA signing

One Ed25519 key pair for ALL Marc's apps — private half in Keychain (label `ed25519`) and 1Password.
Sign DMGs with `Scripts/sparkle-tools/sign_update <path-to-dmg>` from the jellybar source repo.
The tool outputs `sparkle:edSignature="..." length="..."` for the appcast `<enclosure>` tag.
