---
name: bulk-commit-clustering
description: Use when the user has a pile of uncommitted git changes from a long session and asks to organize/commit them in bulk — e.g. "bunch of changes piled up, commit these", "haven't committed in a while, group and commit this", "batch commit my working tree", "cluster these into logical commits". Inspects all changes, detects the project's commit convention, groups files into semantic clusters (one per feature/fix/chore), proposes clusters + messages for approval, then commits each. Do NOT use for an ordinary single commit, "commit and push", or one already-scoped change.
---

# Bulk Commit Clustering

## When this applies

Trigger only when the user is asking you to deal with an *accumulated, mixed pile* of
uncommitted work — not a single change, not a routine "commit and push." Signs it applies:
they mention having skipped commits for a while, ask you to "group," "cluster," "organize,"
or "batch" changes, or describe several unrelated things they built in one sitting that all
need to land as separate commits.

If someone just asks you to commit the thing you *just* did, that's a normal commit — skip
this whole workflow and just commit it.

## Why this workflow exists

Committing after every tiny edit breaks flow during fast, iterative building. But skipping
commits entirely loses the ability to review, bisect, revert, or understand history later.
This skill is the bridge: it lets the user build freely and then, at a natural checkpoint,
turn the pile into the atomic commits they would have made along the way — grouped by what
actually changed *semantically*, not by which directory a file happens to live in.

Because this rewrites how the project's history will look, always show your plan and get
approval before touching git. Never push — this skill's job ends at local commits.

## Step 1: Survey the damage

Before drafting anything, understand the current state:

```bash
git status --porcelain=v1
git diff --stat
git diff --cached --stat
git log --oneline -30
```

- `git status` tells you every tracked-but-changed, staged, and untracked file.
- `git log -30` is how you learn the project's real commit convention — don't assume.
- If the tree is clean (nothing to commit), say so and stop.
- If you're on a detached HEAD, mid-merge, mid-rebase, or otherwise in a weird git state,
  stop and tell the user — don't try to commit through it.

### Detect the commit message convention

Scan the recent log for a pattern before deciding on a style:

- If most recent messages follow `type(scope): summary` (feat, fix, chore, docs, refactor,
  test, etc.) — that's Conventional Commits. Match it, including whether this project tends
  to use scopes.
- If recent messages are short imperative sentences with no prefix ("Add retry logic to
  upload handler") — match that instead. Don't impose Conventional Commits on a project that
  doesn't use it.
- If the log is empty, tiny, or too inconsistent to read a pattern from, default to
  Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`) since it's
  a reasonable, widely-understood default.
- Check for a `CONTRIBUTING.md`, `CONVENTIONS.md`, or similar doc that states a convention
  explicitly — that takes precedence over inferring from history.

## Step 2: Understand every change

For each changed/untracked file, look at the actual diff content, not just the filename —
file paths are a weak signal for what a change *is*:

```bash
git diff -- <file>          # unstaged changes
git diff --cached -- <file> # already-staged changes
cat <file>                  # for new untracked files
```

Read enough of each diff to answer: what problem does this change solve, and what other
changes does it depend on or relate to? A change to a shared util and the three call sites
that now use it belong together even though they're in different folders. Two unrelated
one-line tweaks in the same file do not belong together just because they share a filename.

Watch for:
- **Files with multiple unrelated hunks.** If a single file mixes two unrelated changes
  (e.g. an unrelated formatting fix alongside a real feature edit), don't force the whole
  file into one cluster — note the split explicitly in your plan and use `git add -p`-style
  hunk staging when you commit that cluster (see Step 4).
- **Renames/deletions** — `git status` shows these; they usually belong with whatever change
  caused them.
- **Anything that looks like a secret** — `.env`, `*.pem`, `*_key`, credentials files, etc.
  Flag these clearly in your plan and ask explicitly before including them in any commit;
  don't silently bundle them in.
- **Pure formatting/lint/dependency-lockfile churn** — these are fine to cluster together as
  their own `chore`, separate from behavior changes, unless they're incidental to a specific
  feature (e.g. formatting only the file you just wrote).

## Step 3: Propose clusters and get approval

Group the full set of changes into clusters, where each cluster is what would have been one
atomic commit if the user had committed as they went. For each cluster, work out:

- A short cluster label (e.g. "Add retry logic to upload handler")
- The exact list of files (or file+hunk, for split files) in it
- A one-line rationale — why these belong together
- A drafted commit message in the detected convention. For a cluster with a lot going on,
  it's fine (and often better) to write a subject line plus a few body bullet points rather
  than cramming everything into the subject.

Present the whole plan to the user as a numbered list of clusters before running any git
command that changes state. Something like:

```
1. feat(upload): add retry logic to upload handler
   files: src/upload/client.ts, src/upload/retry.ts, src/upload/client.test.ts
   why: retry.ts is new, client.ts calls it, tests cover the new path

2. chore: bump lockfile after dependency bump
   files: package-lock.json
   why: incidental to nothing else, standalone housekeeping

3. fix(auth): correct token refresh race condition
   files: src/auth/session.ts
   why: unrelated bugfix, independent of the upload work above
```

Wait for the user's go-ahead (or corrections) before proceeding to Step 4. If they want files
moved between clusters, split, or merged, adjust and re-confirm the plan rather than guessing.

## Step 4: Commit cluster by cluster

Once approved, work through the clusters one at a time, most-independent-first is fine, but
keep it simple and sequential:

For a normal cluster:
```bash
git add <file1> <file2> ...
git status                 # sanity check: only this cluster's files are staged
git commit -m "type(scope): subject" -m "optional body bullet 1" -m "optional body bullet 2"
```

For a cluster involving a split file (mixed hunks):
```bash
git add -p -- <file>       # stage only the relevant hunk(s) for this cluster
git status
git commit -m "..."
```

After each commit, re-run `git status --porcelain` before moving to the next cluster so you're
always staging against the current real state, not a stale mental model — especially
important after `git add -p`, which can leave part of a file still unstaged for a later
cluster.

Never run `git add -A` or `git add .` — always stage explicit paths (or explicit hunks) per
cluster, so a file never ends up in the wrong commit or gets swept in by accident.

Never push, and never run `git commit --amend`, `git reset`, `git rebase`, or anything that
rewrites existing history — this skill only adds new commits on top of what's already there.

## Step 5: Summarize

When all clusters are committed, show the result:

```bash
git log --oneline -<N>   # N = number of clusters just committed
git status                # should be clean, or show only what wasn't included
```

Tell the user what got committed as what, and call out anything you deliberately left out
(e.g. a flagged secret file, or a file they said to hold back).
