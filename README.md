# Bench Queue

A small [Claude Artifact](https://www.claude.com/claude-code) that lets a team submit tasks and new-project
requests through a web form instead of editing a `todo.md` file directly. A
recurring background job (run from a Claude Code session) pulls what's
submitted into `todo.md`, commits, and pushes to your repo.

This is a **starter kit** — you publish your own copy under your own
claude.ai account, pointed at your own repo and conventions. Nothing here
talks to any existing deployment.

## Contents

- [`bench_queue_template.html`](./bench_queue_template.html) — the artifact
  itself. Edit the `BENCH QUEUE — CONFIG BLOCK` near the top of its
  `<script>` tag before publishing.
- [`cron_prompt_template.txt`](./cron_prompt_template.txt) — the recurring
  sync-tick prompt, with placeholders for your paths/URLs.
- [`SETUP.md`](./SETUP.md) — prerequisites, the required `todo.md`
  convention, and known limitations.
- [`HOWTO.md`](./HOWTO.md) — step-by-step: the actual prompts to give
  Claude to build and publish this artifact from this repo.

Start with **[HOWTO.md](./HOWTO.md)**.
