# Bench Queue — Setup Guide

Bench Queue is a small Claude Artifact (a private web app hosted on claude.ai)
that lets a team submit tasks and new-project requests through a form instead
of editing a `todo.md` file directly. A recurring background job — run from a
Claude Code session — pulls what's submitted into `todo.md`, commits, and
pushes to your repo. It was built and battle-tested against one real
`todo.md`-in-git workflow; this kit generalizes it so you can stand up your
own independent copy against your own repo and conventions.

This is a **starter kit**, not a hosted product: you publish your own copy,
under your own claude.ai account, pointed at your own repo. Nothing here
talks to the original deployment.

## What you get

- `bench_queue_template.html` — the artifact itself (3 panes: Tasks,
  Deliverables, Projects; a sidebar-style pane bar; light/dark theme).
- `cron_prompt_template.txt` — the recurring background-sync prompt, with
  placeholders for your paths/URLs.
- This guide.

## 0. Prerequisites

- **Claude Code**, with a plan that supports Artifacts (org/team account —
  Artifacts with a shared database are org-internal, never public).
- A **git repo** you can push to, containing (or willing to adopt) a
  `todo.md` that follows this convention:

  ```markdown
  ## <ProjectName>
  **Lab: <TeamOrLabName>**
  **Date:** <any date>

  ### Task List

  #### <short title>
  **Task:** <what to do>
  * <optional sub-bullet>

  ### Deliverables

  \`\`\`yaml
  deliverables:
    - task: |
        <what was asked>
      work_done: |
        <what was actually done>
      status: '<free-text status>'
      date: '<YYYY-MM-DD>'
  \`\`\`
  ```

  (The Deliverables block uses YAML instead of a markdown table on purpose —
  see the note at the top of this kit's parent repo's `todo.md` for why: a
  markdown table breaks if any `work_done` prose contains a literal `|`.)

- A folder of project scaffolding templates, if you want the "New Project"
  form's Template dropdown to mean anything (optional — one generic option
  works fine if you don't have this).

## 1. Publish your own copy of the artifact

1. Open `bench_queue_template.html`, find the block marked
   `BENCH QUEUE — CONFIG BLOCK` near the top of the `<script>` tag.
2. Edit, at minimum:
   - `PROJECTS` — every project name exactly as it appears in your
     `todo.md`'s `## <ProjectName>` headers.
   - `PROJECT_LAB` — each project's `**Lab: X**` value.
   - `LABS` — the distinct list of labs/teams (populates the Lab dropdown).
   - `TEMPLATES` / `DEFAULT_TEMPLATE` — your own project-scaffold template
     names, if you have any; otherwise leave one generic entry.
   - `CATEGORIES` and `tagForProject()` — the tag columns shown on the
     Projects board, and the rule that sorts a project into one. The shipped
     example buckets by name prefix then lab; replace with whatever
     taxonomy makes sense for your team (or collapse to a single category
     if you don't need this at all).
3. In a Claude Code session with Artifact access, ask Claude to publish that
   HTML file as a new artifact with `capabilities: {db: {}}`. Claude will
   hand you back a `claude.ai/artifact/...` URL — that's your Bench Queue.
4. From the artifact's page, use the **Share** menu to give teammates at
   least "Can interact" access (view-only can't submit or check anything).

## 2. Understand the sync direction

**Your `todo.md`/git repo is the single source of truth.** The artifact's
database is a pure inbox: things submitted through the form sit there only
until the next sync pulls them into `todo.md` and deletes them from the
artifact. Don't treat the artifact as a second permanent record — if you
want history, look at `git log`, not the artifact's database.

## 3. Set up the recurring sync

Bench Queue doesn't sync itself — a Claude Code session has to run the sync
logic on a schedule. Two options, in order of reliability:

- **In-session recurring job** (what this kit was built and tested with):
  in a live Claude Code session, ask Claude to schedule a recurring job
  (every ~5 minutes) using the prompt in `cron_prompt_template.txt`, with
  the placeholders filled in. **Caveat:** this job is tied to that one
  session — it stops if the session closes, and expires automatically
  after about a week either way. You (or whoever's driving that session)
  need to re-arm it periodically.

- **Real system crontab:** we tried this and it doesn't currently work —
  the `Artifact`/`ArtifactData` tools needed to read the artifact's
  database aren't available to a headless `claude -p` CLI invocation, even
  under the same authenticated account. A system cron can reliably do
  git/file work headlessly, but can't reach the artifact's data. If that
  changes in a future Claude Code release, the in-session prompt in
  `cron_prompt_template.txt` should still work verbatim from a headless
  invocation — worth re-testing occasionally.

Fill in `cron_prompt_template.txt`'s placeholders before using it:
- `{{ARTIFACT_URL}}` — your published artifact's URL.
- `{{TODO_PATH}}` — absolute path to your `todo.md`.
- `{{REPO_PATH}}` — absolute path to your local git clone that gets pushed.
- `{{GIT_ATTRIBUTION}}` — your commit-footer convention, if any (or delete
  those lines).

## 4. Known limitations to plan around

- **Hardcoded config, not live-read.** `PROJECTS`/`PROJECT_LAB`/`LABS` are
  baked into the published HTML — they don't update themselves when your
  `todo.md` changes. The cron prompt template includes an optional
  reconciliation step (Part B) that diffs `todo.md` against the artifact
  and republishes automatically when they drift; keep it if you want this
  handled, drop it if you'd rather manage the list by hand.
- **No approval gate beyond "someone has to say start it."** The `#top`/
  `#low` priority tags are a labeling convention, not a permissions system —
  anyone with "Can interact" access can tag something `#top`. Add your own
  process around that if multiple people can submit.
- **Org-only sharing.** Artifacts with a database can't be shared publicly
  or with people outside your organization — every user needs an account
  in the same org as the artifact's owner.
- **Browser-only database access from the page itself.** The published
  page can read/write its own database from any signed-in viewer's browser,
  but nothing outside a live Claude session (no plain webhook, no external
  script) can call into it directly today.
