# Slim mirror of `K-Dense-AI/scientific-agent-skills`

A GitHub-importable, auto-refreshing mirror of the upstream skill repo with the
oversized assets removed.

## Why this exists

Upstream cannot be imported as a skill source directly:

| part of upstream | size | needed for skills? |
| --- | --- | --- |
| `docs/` (site assets, incl. `k-dense-web.gif`) | 239 MB | no |
| `.git` history | 223 MB | no (not in a zipball) |
| `skills/` — 164 skills, 2044 files | 23.8 MB | **yes** |
| `tests/`, root metadata (`plugin.json`, …) | 3.2 MB | harmless |

The import pulls a zipball of the working tree — ~272 MB upstream, which trips
the 100 MB limit. Strip `docs/` plus stray image/video files and the tree is
**34 MB**, so a normal GitHub import and refresh works.

Note that only 1.6 MB of imagery lives under `skills/` (4 files in
`timesfm-forecasting/examples/`). Practically all the weight is `docs/`.

## Setup (once, ~3 minutes)

1. Create a **new empty repo** on your GitHub account, e.g.
   `scientific-agent-skills-slim`. Public or private both work.

   > Use a *new repo*, **not a fork**. GitHub disables scheduled workflows in
   > forked repositories, so a fork would never auto-refresh.

2. Add `.github/workflows/sync-slim-mirror.yml` to it, and push. If you want to
   keep this explainer in the repo too, put it at `.github/MIRROR-README.md` —
   **not** at the repo root, which the sync overwrites with upstream's
   `README.md`. Everything under `.github/` survives each sync.

3. Run the workflow once by hand: **Actions → Sync slim mirror → Run
   workflow**. It fetches upstream, deletes `docs/` and any image/video files,
   and commits the result to `main`.

4. Point Claude Science's GitHub skill import at your mirror's URL
   (`https://github.com/<you>/scientific-agent-skills-slim`).

From then on the mirror re-syncs daily at 06:17 UTC, and the platform's normal
"refresh from repository" picks up upstream changes. To pull an upstream change
immediately, hit **Run workflow** and then refresh.

## What the workflow does

Each run replaces the mirror's tree wholesale with upstream's, minus:

- the `docs/` directory
- any `*.png *.jpg *.jpeg *.gif *.webp *.svg *.mp4 *.mov *.pdf *.ico`

It commits on top of the mirror's own history (no force-push), and stripped
files are removed *before* the commit, so large blobs never enter the mirror's
history and the repo stays small over time. If upstream hasn't changed, the run
is a no-op. The run summary prints the skill count and tree size.

Nothing is rewritten inside `SKILL.md` files. Six of them contain relative
links into `docs/`; those links go dead in the mirror, which affects nothing at
skill-load time — the skill bodies are self-contained.

## Upkeep

- **Scheduled runs pause after 60 days of repo inactivity.** GitHub emails you
  and offers a one-click re-enable; `workflow_dispatch` always works.
- **Skill-name collisions.** Three upstream skills share a name with something
  already in your catalog: `diffdock`, `literature-review`, `scvi-tools`.
  Expect the import to either shadow or skip these — check which after your
  first import and decide per skill.
- **Pinning upstream.** To freeze on a known-good upstream commit, change
  `UPSTREAM_BRANCH` in the workflow to a tag or SHA and remove the `schedule:`
  block.

## Alternative: no mirror repo

If you'd rather not host a mirror, `refresh-slim-zip.sh` produces the same
stripped tree as a local zip (~7 MB) for manual import. You re-run it and
re-import whenever you want to update — which is the manual loop the mirror is
designed to avoid.
