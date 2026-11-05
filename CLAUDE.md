# CLAUDE.md — how we work on BDPL LabTools: Dedupe

This file holds the working rules for this application, inherited from the BDPL LabTools family. It is loaded
automatically at the start of every Claude session in this repository, and it
is written for people too: a new contributor, human or AI, should be able to
read it and work the way the lab works. **When a working rule changes, update
this file.**

**The rules shared by every BDPL LabTools repository are one block**, between
the marker lines `BDPL-LABTOOLS-SHARED-RULES:BEGIN` and `:END` below,
byte-identical to the hub's copy in `sul-dlss/bdpl-labtools` `CLAUDE.md`.
Never edit between the markers here: a shared rule changes in the hub first,
and a sync pull request opened from the hub carries it here. The "Rules
drift" check on every pull request is red while the block differs from the
hub's. Everything outside the markers is this repository's own.

These rules were carried over from the BDPL LabKit (`da-capture-macos`),
where each one was learned the hard way. The dates and incident notes are
kept on purpose: a rule with its reason survives; a bare rule gets argued
away.

## What this project is

Dedupe, one application in the BDPL LabTools family built for digital
archivists in the Born-Digital Preservation Lab at Stanford Libraries. Its
job: identify duplicate files within and across accessions so archivists
can decide what to keep, flag or remove before transfer. The family's hub,
catalog and shared conventions live in `sul-dlss/bdpl-labtools`; this file
carries the hub's shared rules block plus the app-specific section at the
end.

## Who merges — the current phase

**Since 2026-11-01, every pull request needs one approving review from a
collaborator other than its author, and a code owner's approval, before it
can merge.** Sally DeBauche (`SallyDeBauche`) and Annie Schweikert
(`aeschweik`) joined with write access that day and the "Protect main"
ruleset was raised to match: one required approval, code-owner review,
approval of the last push, review threads resolved, no bypass for anyone.
Michael Olson (`mgolson-stanford`) is the code owner and remains the merge
gate for anything he did not author; a PR of his own needs a colleague's
approval first.

The solo phase (2026-11-01, zero required approvals while the scaffolding
PRs landed) is over. Claude never approves and never merges.

<!-- BDPL-LABTOOLS-SHARED-RULES:BEGIN — shared by every repository in the family. Source: sul-dlss/bdpl-labtools CLAUDE.md. In an app repository, do not edit between these markers; change the hub and run tools/sync-rules.py. -->

## Project-management files (keep these current)

- **`BACKLOG.md`** — the single prioritized backlog of **open** work. New
  ideas and deferred work go here with a priority and a one-line rationale.
  When an item ships, move it verbatim to `BACKLOG-done.md` with its
  "Shipped:" note. `BACKLOG.md` holds no checked-off items: a list that
  mixes work with history stops reading as a list of work.
- **`PROGRESS.md`** — the historical record: session summaries, decisions
  and rationale, tool inventory, bugs found. Append; never rewrite history.
- **`docs/README.md`** — the index of `docs/`. Add a line when you add a
  document and keep the status column honest: an executed plan and a parked
  plan look identical from the filename, and reading a parked plan as current
  is how something gets built twice.
- **`.claude/agents/`** — agent charge documents. `docs-steward` runs on every
  PR; add agents by PR and list them in `.claude/agents/README.md`.

Typical flow: pick an item from `BACKLOG.md` → **create a feature branch** →
build and verify → **open a pull request** → **Michael approves and merges** →
log the outcome in `PROGRESS.md` → move the item to `BACKLOG-done.md`.

## Working rules

1. **Verify on real behavior, not just syntax.** Test a change by driving the
   actual code path: module tests, a throwaway instance, real hardware when
   safe. Validate claims with independent tools where possible (bags with the
   Library of Congress `bagit.py`, hashes against `shasum`).
2. **Never write to source media.** In any LabTools app, sources are only
   block-read (imaging) or mounted read-only (logical capture). Anything that
   could touch a source device gets extra scrutiny.
3. **Don't disrupt the user's running instance.** Test on separate throwaway
   instances; if a shared config value is changed during testing, reset it.
4. **No new runtime dependencies without a reason in the PR.** Prefer the
   standard library. External *tools* (libewf, ClamAV, Siegfried,
   Brunnhilde and the like) are fine but must be detected gracefully and
   documented.
5. **Least privilege.** Root is for raw-device reads only; everything else
   must work without it. Local web interfaces bind `127.0.0.1` only, behind a
   per-session token. Don't add new privileged paths.
6. **Update docs with behavior changes**, in the same pull request. Keep any
   log or metadata mapping aligned with the lab's templates and controlled
   vocabularies, verbatim. **Every pull request runs the `docs-steward`
   agent before it is offered** (`.claude/agents/docs-steward.md`): it
   reads the diff, fixes the documents the change makes stale on the same
   branch, keeps `docs/README.md` (and, in the hub, the catalog) honest, and
   writes its report into the PR body under "Docs steward". A PR offered
   without that section is not ready. (Michael's rule, 2026-11-01.)
7. **Git workflow (mandatory — no exceptions).**
   - **Never commit or push directly to `main`.** All work happens on a
     branch (`feat/…`, `fix/…`, `docs/…`).
   - Changes reach `main` **only via a pull request** on GitHub.
   - Every PR is merged by Michael (see *Who merges*). Claude never merges,
     and never treats silence as approval.
   - **Claude never closes a pull request or an issue**, not a superseded
     one, not a stale one, not a duplicate. Closing is a disposition act and
     is Michael's, exactly like merging. A superseded PR stays open, gets a
     comment naming its replacement, and its own line in the list handed to
     Michael: "superseded by #N; close it if you agree." (Michael's rule,
     2026-11-03: two rebuilt PRs were closed as housekeeping and he found
     them gone when he went to merge.)
   - **Do not circumvent this rule**: not to save time, not because a change
     seems trivial, not for any reason. No direct pushes, no force-pushes to
     `main`, no admin overrides, no self-merges by Claude. If the workflow
     seems to block progress, stop and ask Michael.
   - **An offered PR is SEALED.** Once Claude tells Michael a PR is ready to
     merge, nothing more is pushed to that branch; new work gets a new PR.
     (Michael merges fast; a push racing his click gets silently orphaned on
     a closed PR's branch. It happened twice in LabKit, #288 and #310, both
     caught by the four-count check below and rescued by cherry-pick PRs.) If
     an addition to an open PR is truly needed, say "hold the merge — one
     more commit coming" BEFORE pushing, and wait for his acknowledgement.
   - **An offered PR comes with its link.** When Claude tells Michael a PR
     is ready, the message carries a direct link to the PR's GitHub page
     (for example `https://github.com/sul-dlss/bdpl-labtools/pull/3`), with
     its title and CI state, so he can go straight to the merge button.
     Several PRs are listed in the order to merge them. (Michael's rule,
     2026-11-01.)
   - **Check CI before opening a PR and again before it merges.** Run the
     linters locally and read `gh pr checks <n>` rather than assuming.
     **A red PR is not ready to merge**: say so, or fix it, but do not hand
     it over silently.
   - **Verifying a merge means verifying what it carried.** After a merge,
     confirm via the API that it happened, that it took the branch's real
     head, that CI on `main` passed for that merge commit, and that one
     distinctive string from the diff is present on `main`. That is the
     four-count check. *Learned 2026-08-08 in LabKit: seven consecutive
     merges to `main` ran red for two and a half hours. Every merge had been
     "verified" for landing and for taking the right head, and nobody looked
     at the checks. One finding was a `NameError` on the one exit path an
     operator actually uses. A half-check that reads as thorough is worse
     than no check, because it stops anyone looking further.*
   - **Releases are tags.** Production runs a release tag cut from green
     `main`; tagging is Michael's act, like merging. `git pull` is never a
     deploy; rollback is the previous tag.
   - Hygiene: repo-local git identity set to the contributor's GitHub
     no-reply address; descriptive commit messages; imaging outputs
     (`*.dd`, `*.E01`, `*.iso`, imaging logs) and collection data are
     gitignored and never committed.
8. **Ask before destructive or irreversible actions**: deleting user data,
   changing global config, rewriting pushed history, installing system-level
   software. Interactive installers are run by the user; Claude verifies.

## Done means — the four-line checklist

Before a pull request is offered, all four; whichever is skipped is named
in the PR body.

1. **The suite and the linters are green locally.** CI is the second look,
   not the first.
2. **Every user-facing surface still works.** For a web app, every page
   loads with no script error and no failed resource on a throwaway server.
3. **The handed artifact was exercised.** The exact link, command, button or
   file the PR hands over was driven once, not just its machinery. A
   browser-driven change shows its screenshot in the PR; never a token, key
   or passphrase in a picture.
4. **The record says what was NOT verified.** The PR body names what was
   proven by reading rather than by driving, and confirms the docs moved
   with the behavior.

## Security baseline (already in place on this repo)

Secret scanning with push protection; Dependabot alerts and security
updates; private vulnerability reporting; read-only workflow tokens; CI
running ruff, shellcheck, bandit and gitleaks on every PR; the "Protect
main" ruleset. New `bdpl-labtools-<app>` repositories get the same set on
creation. See `SECURITY.md` and `CONTRIBUTING.md`.

<!-- BDPL-LABTOOLS-SHARED-RULES:END -->

## Dedupe-specific (fill in as decisions are made)

- **Inputs** are read-only. Dedupe never modifies, moves or deletes files
  in an accession; it reports. Any action on duplicates is a separate,
  explicit, logged step taken by the archivist.
- **Hashes are the evidence.** Record the algorithm and the full digest
  with every match; a report that says "duplicate" without the hash is not
  a report.
- **Platform decisions** (macOS-only calls, file-system quirks) are logged
  in `docs/portability-log.md` in the same change that adds them, as in
  LabKit.
- Technical constraints, tool choices and the data model go here as they
  are decided, each with its date and reason.
