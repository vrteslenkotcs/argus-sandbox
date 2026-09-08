# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repo is a **sandbox for testing the `taiga-family/argus` GitHub Action** — a screenshot-diff bot that comments on PRs with visual regression results. There is no application code, no build, and no test runner. Everything here exists to drive the Argus action through realistic CI scenarios.

## How the simulation works

The CI is wired to fake an E2E run, so changes are validated by opening a PR and watching the bot, not by running anything locally.

- `.github/workflows/e2e.yml` — pretends to be the E2E job. It uploads `snapshots/**/*.png` as an artifact named `snapshots`, then `exit 1` to force a failed status. **The `exit 1` is intentional** — Argus expects a failed E2E run to react to. Don't "fix" it.
- `.github/workflows/screenshot-bot.yml` — triggers on `workflow_run` of the E2E job (and on `pull_request: closed`). It runs `taiga-family/argus@main`, which downloads the `snapshots` artifact, classifies images, and comments on the PR.
- `.github/screenshot-bot.config.yml` — `screenshotsDiffsPaths` is a list of regexes; files matching them are treated as **failing diffs**, everything else as passing snapshots.

`snapshots/` is the fixture set fed to the bot. By the current config (`.*.diff.png`):
- `datalist.diff.png` → reported as a diff
- `not-diff-snapshot.png`, `another-not-diff-snapshot.png` → reported as passing

To exercise a new bot scenario, change the fixtures or the config regex and open a PR — do not try to run the workflow locally.

## Branch exclusions

`screenshot-bot.yml` skips branches matching `v[0-9].x`, `release/**`, and `debug-github-app`. Most commit history in this repo is iterating on which `branches-ignore` patterns and action refs work — when adjusting either, check recent commits first to see what's already been tried.