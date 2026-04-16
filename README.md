# cheap-merge-queue

A ~30-line GitHub Action that mimics GitHub's Enterprise-only merge queue on Teams/Free plans.

## What it does

On every push to `main`, walks all open PRs targeting `main` that have
**auto-merge enabled** and calls the REST "Update branch" endpoint on each.
PRs without auto-merge are left alone (they're still being iterated on). PRs
with conflicts are skipped (logged, not failed).

Using auto-merge as the signal is the point: the author has explicitly said
"this is ready — land it when the gates open." Everything else is in-progress
and shouldn't be churned.

Combined with the branch protection rule **"Require branches to be up to date
before merging"**, this is effectively a serial merge queue: no PR can land
against a stale `main`.

## Usage (copy/paste approach)

1. Copy `.github/workflows/auto-update-prs.yml` into your repo.
2. Enable branch protection on `main`:
   - Require status checks to pass
   - **Require branches to be up to date before merging** ← this is the one
3. Done.

## Why not use GitHub's built-in merge queue?

It's Enterprise-only. This file is free.

## Limitations vs. a real merge queue

- No batching (can't test A+B+C together to land them faster).
- Tiny race window: if A merges and B's merge button is clicked within ~seconds
  before the workflow fires, B could land stale — but the "up to date" branch
  protection rule rejects that.
- Uses the default `GITHUB_TOKEN`, so pushes from this workflow don't trigger
  other workflows (GitHub's loop-prevention). If you need downstream CI to
  re-run on the updated branch, swap in a PAT or GitHub App token.
