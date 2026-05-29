---
title: "Generate CHANGELOG entry from commits"
category: docs
when_to_use: "When cutting a release and you need a CHANGELOG.md entry that summarises commits since the previous tag"
tested_with: ["Claude Opus 4.7", "Claude Sonnet 4.6", "GPT-5"]
version: 1.0
---

## Description

Turns a list of commit messages into a Keep-a-Changelog-formatted entry,
grouped by audience-facing change type. The non-obvious move is forcing
the model to drop commits that do not change user-visible behaviour
(internal refactors, dependency bumps, doc-only changes) unless explicitly
asked to include them. Most CHANGELOGs are noisy precisely because they
include too much.

This prompt is stack-agnostic — it works on any commit list, in any
language, regardless of whether the team uses Conventional Commits.

## Prompt

```
Generate a CHANGELOG.md entry for version <VERSION> from this commit
range. Follow the Keep a Changelog format (https://keepachangelog.com).

Group entries under these headings, in this order, skipping any that
would be empty:

  ### Added         — new user-visible features
  ### Changed       — behavioural changes to existing features
  ### Deprecated    — features still present but slated for removal
  ### Removed       — features that no longer exist
  ### Fixed         — bug fixes visible to users
  ### Security      — security-relevant fixes (always include if any
                       exist, even patches)

Per entry rules:
  - One line per change, present tense, user perspective: "Resolves an
    issue where uploads over 50 MB hung indefinitely" — not "Fixed a bug
    in the upload service".
  - Cite the PR or commit hash in parentheses at the end.
  - Group multiple commits that delivered one user-visible change into a
    single line.
  - Drop commits that do not change user-visible behaviour:
    refactors, test additions, internal docs, CI changes, dependency
    bumps (unless the bump fixes a security CVE or is otherwise
    user-relevant). List them under "Internal (not in changelog)" so
    nothing is silently dropped.

Output:
1. The CHANGELOG entry, ready to paste at the top of `CHANGELOG.md`
   under the existing entries.
2. "Internal (not in changelog)" — commits you deliberately excluded,
   one line each with the reason.

Constraints:
- Do not include a "Breaking Changes" heading unless the project's
  existing CHANGELOG uses it. Keep a Changelog spec puts breaking changes
  under their respective Changed / Removed sections, marked **(breaking)**.
- Do not invent change descriptions when a commit message is unclear.
  List the unclear commit at the end with "needs author clarification".
- If the version is a major bump (X.0.0), surface breaking changes at the
  top of the entry in a short prose paragraph before the headings.

Commits since previous tag:
<paste `git log <prev-tag>..HEAD --oneline` output, or full log with
bodies if available>
```

## Example

**Input:** 18 commits between v0.4.0 and v0.5.0.

**Expected output (excerpt):**

> ## [0.5.0] — 2026-05-22
>
> ### Added
> - Address autocomplete in the checkout form (#221).
> - Export project data to CSV from the project settings page (#229).
>
> ### Changed
> - Project list now paginates at 25 items per page instead of loading
>   all projects up front (#225). **(breaking)** API responses no longer
>   include a top-level `total` field; clients should use the new
>   `pagination.total` field instead.
> - Stripe webhook retries now back off exponentially up to 5 attempts,
>   previously fixed 3-attempt retry (#231).
>
> ### Fixed
> - Resolves an issue where uploads over 50 MB hung indefinitely on
>   slow connections (#226).
> - Avatar image no longer renders as a broken icon for users without an
>   uploaded avatar (#228).
>
> ### Security
> - Address CVE-2026-12345 in `next` by upgrading to 14.2.5 (#234).
>
> **Internal (not in changelog):**
> - `chore: bump biome to 1.8` (#220) — internal tooling
> - `refactor: split user controller into 3 files` (#222) — no
>   user-visible change
> - `test: add coverage for password reset` (#223) — tests only
> - `docs: clarify deployment guide` (#224) — internal docs
> - `ci: cache pnpm store` (#227) — CI only
> - `chore: bump dependencies (patch)` (#230, #232, #233) — patch bumps
>   with no user-relevant fixes
>
> **Needs author clarification:**
> - `#235 wip` — commit message too vague.

## Personalisation tips

- **Conventional Commits:** if the project uses Conventional Commits,
  prepend "Trust the commit type prefix (feat:, fix:, etc.) but verify
  against the body before grouping." Output quality jumps substantially.
- **No squash:** for projects with many small commits per PR, work from
  PR titles instead — append "Use the PR title as the source of truth,
  not individual commits, and ignore squashed individual commit messages."
- **Customer-facing vs internal:** generate two outputs in one pass —
  "Also generate a `customer_changelog.md` entry, ≤80 words, no PR
  numbers, no internal terminology."
- **Date format:** Keep a Changelog uses `YYYY-MM-DD`. If your project
  uses something else, state it explicitly.
- **Multiple releases at once:** if you're catching up on several missed
  releases, run the prompt once per version range. Do not try to
  generate multiple entries in one pass; quality drops.
