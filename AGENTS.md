# AGENTS.md

## Project overview

`dockhide` — a single Bash script that hides a macOS app's Dock icon by setting `LSUIElement` in its `Info.plist` and ad-hoc re-signing the bundle, and can restore it later.

## Setup

No install step. macOS only; uses system tools (`/usr/libexec/PlistBuddy`, `codesign`, `lsregister`).

## Build / Run

```sh
./dockhide help
```

## Test

No automated test suite. Exercise it against a throwaway copy of an app rather than a real install:

```sh
mkdir -p /tmp/dh && cp -R /System/Applications/Calculator.app /tmp/dh/
./dockhide hide /tmp/dh/Calculator.app && ./dockhide status /tmp/dh/Calculator.app
./dockhide show /tmp/dh/Calculator.app && codesign -v /tmp/dh/Calculator.app
```

`shellcheck dockhide` must pass (CI runs it).

## Repository structure

- `dockhide` — the entire tool. `VERSION` is bumped by release-please (`extra-files` in `release-please-config.json`), so don't edit it by hand.
- `.github/workflows/release-please.yml` — maintains the release PR; runs with the `RELEASE_PLEASE_TOKEN` PAT so the tag it pushes triggers `release.yml`.
- `.github/workflows/release.yml` — on a `v*` tag, uploads `dockhide` as a release asset and opens a PR on `zackwag/homebrew-tap` updating `Formula/dockhide.rb`'s url and sha256 (needs the `TAP_TOKEN` PAT).

## Design notes

- `hide` records the app's original `LSUIElement` value (`unset`, `true` or `false`) in a `DockhideOriginalLSUIElement` key; `show` restores from it and `list` finds apps by it. Apps without the marker are never modified by `show`.
- Each app argument runs in its own subshell with `set -e` re-enabled; don't switch that to `( ... ) || rc=1`, which silently disables `set -e` inside.

## Commit and PR conventions

- Commit messages and PR titles must follow [Conventional Commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`, `ci:`, `build:`, `perf:`, `style:`, `revert:`), optionally with a scope, e.g. `fix(api): handle null response`.
- This repo squash-merges pull requests only; the PR title becomes the final commit message on `main`.
- A "Conventional Commits" CI check enforces this on both PR titles and direct-push commit messages.
- Branch protection on `main`: no force-pushes, no branch deletion, required status checks must pass.
