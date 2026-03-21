# VoiceInk Sync Workflow

## Repo Facts

- Repo path: `/Users/iancosnaye/developer/utils/VoiceInk`
- Upstream remote: `origin` -> `https://github.com/Beingpax/VoiceInk.git`
- Fork remote: `wordup` -> `git@github.com:word-up/VoiceInk.git`
- Main long-lived customization branch: usually `feat/wordup-internal-build`

## Before Syncing

1. Run:

```bash
git status --short --branch
git remote -v
git branch -vv
git fetch --all --prune
git log --oneline --decorate --graph --max-count=25 --all
```

2. Confirm how the user actually runs the app:
   - If they use personal local/dev builds, preserve `LocalBuild.xcconfig` compatibility.
   - If they truly need an internal-distribution build, keep any special team/bundle-id changes only if still required.

3. Check the local-only commit set:

```bash
git log --reverse --format='%h %s' origin/main..HEAD
git diff --stat origin/main..HEAD
```

## Preferred Integration Strategy

Use replay, not blind merge.

1. Create a backup branch from the current working branch.
2. Create a temp branch from fresh `origin/main`.
3. Cherry-pick the local custom commits one by one.
4. Resolve conflicts narrowly. Preserve upstream version bumps and structural changes unless a local customization explicitly needs otherwise.

Example shape:

```bash
git branch backup/<name>-pre-sync-$(date +%Y%m%d-%H%M%S)
git checkout -b codex/<name>-sync origin/main
git cherry-pick <commit-1> <commit-2> ...
```

## VoiceInk-Specific Customizations To Protect

These were the practical customizations retained during the last successful sync:

- Remove trial enforcement:
  - `VoiceInk/Models/LicenseViewModel.swift`
- Traditional Chinese plus bilingual auto-detect prompt:
  - `VoiceInk/Whisper/WhisperPrompt.swift`
- Preserve original language during enhancement:
  - `VoiceInk/Models/AIPrompts.swift`

Treat internal-distribution-only config separately. Do not keep it by default for personal local builds.

## Local Build Data Safety

This repo has a sharp edge: local builds can appear to "lose" prompts, API keys, and settings if the bundle id changes.

### Why this happens

- Local builds use `LOCAL_BUILD`.
- In `LOCAL_BUILD`, `KeychainService` stores secrets in `UserDefaults` under `LocalKeychain_*`.
- `UserDefaults` are namespaced by bundle id.
- If the installed app's bundle id changes, the app reads a different preference domain and your old settings appear missing.

### Required local-build setting

Keep this in `LocalBuild.xcconfig`:

```xcconfig
PRODUCT_BUNDLE_IDENTIFIER = com.ian.VoiceInk
```

This is mandatory when the user expects continuity with their previous local app data.

### Fast recovery checks when data seems missing

Run:

```bash
/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' /Applications/VoiceInk.app/Contents/Info.plist
defaults domains | tr ',' '\n' | rg 'VoiceInk|com\.ian|com\.prakash'
defaults read com.ian.VoiceInk 2>/dev/null | rg 'TranscriptionPrompt|CustomLanguagePrompts|LocalKeychain_|selectedAIProvider|CurrentTranscriptionModel|SelectedLanguage' -n || true
defaults read com.prakashjoshipax.VoiceInk 2>/dev/null | rg 'TranscriptionPrompt|CustomLanguagePrompts|LocalKeychain_|selectedAIProvider|CurrentTranscriptionModel|SelectedLanguage' -n || true
```

Interpretation:

- If `/Applications/VoiceInk.app` is not `com.ian.VoiceInk`, rebuild and reinstall with the local-build bundle id fixed.
- If `com.ian.VoiceInk` contains the old values, the data is not gone; the app is just reading the wrong domain.

## Build And Install

Use this when the user asks for a ready-to-run app:

```bash
pkill -x VoiceInk || true
make local
rm -rf /Applications/VoiceInk.app
ditto "$HOME/Downloads/VoiceInk.app" /Applications/VoiceInk.app
xattr -cr /Applications/VoiceInk.app
open -na /Applications/VoiceInk.app
```

Then verify:

```bash
/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' /Applications/VoiceInk.app/Contents/Info.plist
pgrep -fl '/Applications/VoiceInk.app|VoiceInk' || true
```

## Validation Expectations

- Prefer `make local` or equivalent local-build `xcodebuild` verification.
- A plain Debug build is not enough when local-build data compatibility matters.
- `xcodebuild test` may be blocked by macOS UI test runner security dialogs on this machine. Treat that as an environment limitation, not automatically as a regression.
- If needed, report this clearly instead of claiming full test coverage.

## Final Push Rules

- Rewrite the feature branch only after verification succeeds.
- Force-push only to `wordup`, for example:

```bash
git push --force-with-lease wordup feat/wordup-internal-build:feat/wordup-internal-build
```

- Do not push rewritten history to `origin`.

## Good Final Summary

Mention:

- upstream version reached
- local commits kept
- whether internal-distribution config was removed or preserved
- whether local build was verified
- whether `/Applications/VoiceInk.app` was replaced
- any remaining limitation such as blocked UI tests
