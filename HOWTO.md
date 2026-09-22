# How to build this artifact in Claude, from this repo

This walks through actually *doing* the setup described in `SETUP.md` —
concretely, what to type into Claude to get your own working Bench Queue.
You need [Claude Code](https://claude.com/claude-code) with an account on a
plan that supports Artifacts (Artifacts with a shared database are
org-internal — everyone who'll use this needs an account in the same org).

## Step 1 — Clone this repo

```bash
git clone https://github.com/andrebolerbarros/bench_queue.git
cd bench_queue
```

Start a Claude Code session in this directory.

## Step 2 — Point Claude at your own `todo.md`

Bench Queue assumes a `todo.md` that looks like this (see `SETUP.md` for the
full spec):

```markdown
## <ProjectName>
**Lab: <TeamOrLabName>**
**Date:** <any date>

### Task List
...

### Deliverables

```yaml
deliverables:
  - task: |
      ...
    work_done: |
      ...
    status: '...'
    date: '...'
```
```

If you don't have one yet, write a minimal one first (even 2-3 projects is
enough to start). Then, in your Claude Code session, prompt:

> Read `bench_queue_template.html` in this repo. Find the block marked
> `BENCH QUEUE — CONFIG BLOCK` near the top of its `<script>` tag. Read
> `/path/to/your/todo.md` and update `PROJECTS`, `PROJECT_LAB`, and `LABS`
> in that config block to match it exactly — one entry per `## <Project>`
> header and its `**Lab: X**` value. Leave everything else in the file
> unchanged.

If you want project-scaffold templates in the "New Project" form's
dropdown, also say what folder they live in and their names, and ask Claude
to fill in `TEMPLATES`/`DEFAULT_TEMPLATE` to match.

If you want the Projects board's tag columns to mean something for your
team, describe your own categorization rule and ask Claude to update
`CATEGORIES` and `tagForProject()` accordingly (or ask Claude to just
collapse it to a single category if you don't need this at all).

## Step 3 — Publish it

Once the config block reflects your setup, prompt:

> Publish `bench_queue_template.html` as a new Artifact, with
> `capabilities: {"db": {}}` so it gets a shared database. Give it the icon
> "clipboard".

Claude will hand you back a `claude.ai/artifact/...` URL — that's your Bench
Queue. Open it, then use its **Share** menu to give teammates at least "Can
interact" access.

## Step 4 — Wire up the recurring sync

Fill in `cron_prompt_template.txt`'s placeholders (`{{ARTIFACT_URL}}`,
`{{TODO_PATH}}`, `{{REPO_PATH}}`, `{{GIT_ATTRIBUTION}}`) with your actual
values, then in the same Claude Code session, prompt:

> Schedule a recurring job with `CronCreate`, cron `"2-59/5 * * * *"`
> (every ~5 minutes), using this exact prompt: `<paste the filled-in
> contents of cron_prompt_template.txt here>`

This job only runs while that session stays open, and auto-expires after
about a week (Claude will tell you this when you schedule it) — you'll need
to re-arm it from a live session periodically. See `SETUP.md` §3 for why a
plain system crontab can't currently do this instead.

## Step 5 — Use it

- **Tasks** pane: submit work through the form; check `#top` if it should
  start as soon as the next sync picks it up, leave unchecked (`#low`) to
  just queue it.
- **Projects** pane: browse existing projects by tag, or request a new one
  via the `+` button.
- **Deliverables** pane: whatever Claude has marked done shows up here for
  you to check off — that's a separate manual step from the sync (ask
  Claude to seed/update this collection from your `todo.md`'s Deliverables
  blocks; it's not wired into the automatic sync tick in this kit).

## Iterating later

Any time you want to change the page (add a field, change a color, adjust
the config block), just ask Claude to edit `bench_queue_template.html` and
republish it with the **same URL** (`url: <your artifact's URL>` on the
publish call) — that updates the live page in place rather than creating a
new one.
