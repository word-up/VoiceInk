---
name: voiceink-upstream-sync
description: Update the VoiceInk repo from upstream while preserving this project's local customizations, local-build behavior, and app data. Use when working in `/Users/iancosnaye/developer/utils/VoiceInk` on requests like "pull latest VoiceInk", "sync upstream", "update GitHub version", "rebuild and install VoiceInk", or "my prompt/API key/settings disappeared after update". This skill is specific to the VoiceInk remotes and local build setup in this repo.
---

# VoiceInk Upstream Sync

Use this skill for upstream syncs in this repo. Keep the workflow conservative and data-safe.

## Core Rules

- Treat `origin` as the upstream repo and `wordup` as the fork to push to.
- Never open or push a PR to `origin` unless the user explicitly asks.
- Preserve local-build data compatibility. For this repo, local builds must keep `PRODUCT_BUNDLE_IDENTIFIER = com.ian.VoiceInk` in `LocalBuild.xcconfig`.
- Assume the user may rely on local-only settings and API keys. In `LOCAL_BUILD`, API keys are stored in `UserDefaults`, not stable Keychain storage.
- Prefer replaying local custom commits onto fresh `origin/main` instead of merging a long-lived branch blindly.

## Quick Workflow

1. Inspect git state, remotes, branch, and local changes.
2. Fetch both remotes and confirm what changed in `origin/main`.
3. Identify the true local custom commits to preserve.
4. Create a temp integration branch from fresh `origin/main`.
5. Cherry-pick only the desired local commits.
6. If the user uses personal local builds, drop internal-distribution-only changes.
7. Verify with a local build, not just a generic Debug build.
8. Force-push only to `wordup/<branch>` when rewriting history.
9. If asked to install the app, rebuild and replace `/Applications/VoiceInk.app`.

## What To Read Next

- Read [workflow.md](./references/workflow.md) before doing the sync.
- Re-read [workflow.md](./references/workflow.md) if settings, prompts, API keys, or dictionary data appear missing after install.
