---
layout: post
title: "The Nine Pillars of a Level 4 Agentic Development Setup"
description: "What you need around a coding agent so you can hand over a task and walk away: nine pillars, what each one is, why it matters and how to build it."
---

*I haven't written a line of code in over a year. I still ship software, more than before. This
article describes the setup that makes that possible: nine pillars, what each one is, why it
matters, and how to build it. It is the written, extended version of my talk at IT Days 2026 in
Cluj, and the first article in a series on Pathfinder.*

## In short

- Getting from "AI helps me code" to "agents do the work, I manage them" is not a model problem.
  Today's models are good enough. It is a **setup** problem.
- The setup has nine pillars in two groups. Six build **the agent's workplace**: toolbox
  container, ephemeral environments, guardrails, the wiki, instructions and skills, agentic
  colleagues. Three build **your desk**: spec-driven flow, command center, proof of work.
- If you only take three ideas away: **invest in the plan, demand evidence, and fix the setup
  instead of the code.**

## Scope

**Who this is for.** Developers, tech leads and architects who already use a coding agent (Claude
Code, GitHub Copilot CLI or similar) and want to move from pairing with it to delegating to it.

**What it covers.** The technical and working setup for one developer or one team: where agents
run, what they know, how they are kept safe, and how you plan, follow and check their work.

**What it does not cover.**

- Choosing a model or prompt tricks. The pillars apply to any capable agent.
- The organisation around the team: planning, release trains, budgets. That is the next article.
- Fully autonomous delivery ("Level 5"). I explain below why I don't think it is realistic yet.

**Assumptions.** Your code lives in git, changes go through pull requests, and your project can be
built and tested from the command line.

**How each pillar is described.** *What it is*, *why it matters* (what breaks without it), *how
to build it* (practices that worked for me), *common mistakes*, and where it lives in **Pathfinder
Dev**, the open-source reference implementation I built and use daily.

## Background: the five levels

In January 2026, Dan Shapiro described five levels of AI-assisted development, modelled on the
levels of self-driving cars:

| Level | Name | The human |
|---|---|---|
| 0 | Spicy autocomplete | Writes everything; the AI suggests the next line |
| 1 | The coding intern | Writes the important code; the AI writes boilerplate |
| 2 | The junior developer | Pairs with the AI and reviews every line |
| 3 | The developer | Becomes a full-time code reviewer |
| 4 | The engineering team | Writes specs, manages agents, checks results |
| 5 | The dark factory | Is not in the loop |

Most developers who call themselves AI-native stop at Level 2, because Level 2 feels done. It
isn't. Level 4 is realistic today; it is where I work. Level 5 is not, for most of us: the few
teams near it are small and spend most of their agents' effort proving that the software works.

The step from Level 2 to Level 4 changes your role:

| Before | At Level 4 |
|---|---|
| Write the code | Architect the solution |
| Fix the bug | Coach the agent |
| Do the task | Delegate the task |
| Review my own work | Review my colleagues' work, and my colleagues are agents |

A manager doesn't give a new hire a laptop with nothing installed, no documentation, no access to
the backlog and no way to test, and then expect good work. Yet that is what most agent setups look
like. The nine pillars are what a new colleague gets on day one.

## The model: nine pillars, two groups

| The agent's workplace (where it works, what it knows) | Your desk (how you manage it) |
|---|---|
| 1 Toolbox container | 7 Spec-driven flow |
| 2 Ephemeral environments | 8 Command center |
| 3 Guardrails | 9 Proof of work |
| 4 The wiki | |
| 5 Instructions and skills | |
| 6 Agentic colleagues | |

A task flows through them like this:

```
Plan (you + agent) → Implement (agent) → Test (agent) → Pull request with evidence → Review (you)
```

Two human steps, at the start and at the end. Everything in the middle runs without you, inside
the workplace, and comes back to your desk.

---

## Part 1: The agent's workplace

### Pillar 1 · Toolbox container

**What it is.** One container image with every tool a developer on the project uses, running on
your laptop or on a remote machine.

**Why it matters.** An agent can only do a developer's work with a developer's tools. If it can't
build, run or test, it guesses, and guessing is Level 2. The container is also what lets you run
the agent somewhere else, and that changes your behaviour: when the agent runs on your laptop, you
keep watching the session. When it runs on a server, you start it and stop looking. Out of sight
is what makes you a manager.

**How to build it.**

- Split the image into a **base** (git, the coding agents, language runtimes, Docker CLI, cloud
  CLIs, a headless browser) and a **project layer** (build tools, linters, database clients, MCP
  servers, Playwright) defined in the repository.
- Keep the project layer as **configuration in the repo**, reviewed like code, so the whole team
  gets the same agent.
- If a task needs a tool, **add it to the image**. Don't let the agent work around a missing tool,
  and don't install it by hand in a running container.
- **Start locally.** If nothing depends on the host, moving to a VM is a deployment, not a rewrite.
  Locally you run a few agents while the laptop is on; on a VM, as many as it holds, all night.
- Prefer tools with **fast, readable output**. The agent must see the result of what it did:
  build output, test results, logs, the running app. A fast feedback loop beats a smarter model.

**Common mistakes.** Running the agent directly on your workstation with your credentials. A
"golden image" nobody can change. Tools installed by hand that disappear with the next container.

**In Pathfinder Dev.** A base image with the agents, runtimes and Playwright; `agentic init`
detects the project's tools and writes `.agentic/agentic.yaml`; `agentic build` creates the project
image; `agentic dev` and `agentic run` start it locally or detached.

### Pillar 2 · Ephemeral environments

**What it is.** Every task gets its own isolated environment: its own branch, its own copy of the
app, its database and its dependent services. Started on demand, removed when the work is merged.

**Why it matters.** Agents must test their work independently and in parallel. With one shared dev
environment they overwrite each other's changes, or wait in line. Five agents on five stories need
five environments; otherwise you are back to one at a time.

**How to build it.**

- **One task, one branch, one worktree.** A task that goes wrong is thrown away without touching
  the others.
- Describe the project's services (database, queue, mocks) **once**, for example as a Compose file
  in the repo, and start a private copy per task.
- Avoid fixed ports and shared state. Give each task its own network and random local ports.
- **Seed test data by script**, so every environment starts from a known state.
- Tear everything down automatically when the task ends. Forgotten environments cost money and
  confuse the next run.

**Common mistakes.** Sharing one database "just for now". Environments that need manual steps to
start. Ports hard-coded in the app's config.

**In Pathfinder Dev.** Each detached task can get its own worktree (`agentic run --worktree`), and
the project's Compose services join a per-project network. Per-task stacks, a private copy of the
project's services for each task, are being built now.

### Pillar 3 · Guardrails, not supervision

**What it is.** Rules that hold without a human watching: a sandbox, enforced checks, and a gate
on anything that leaves the sandbox.

**Why it matters.** You can't walk away from something you have to watch. Guardrails replace
supervision, and that is what makes remote agents and scheduled agents safe. Without them, every
other pillar that lets you step away is a risk.

**How to build it.**

- **The container is the sandbox.** The agent can break its own box, not your machine.
- **Hard rules live in code, not in prompts.** A rule in a prompt holds most of the time; a rule in
  a script or a hook holds every time. Let a script decide wherever a script can, and use the model
  only for judgement.
- **Gate outward actions.** Pushing, opening pull requests, posting and deploying pass a check
  first, and are allowed per task, not globally.
- **Branch and pull request only.** Never straight to the main branch.
- **Define when the agent stops and asks:** when a rule says so, when it is unsure, when a check
  keeps failing, or when the budget runs out. A good agent doesn't guess when stuck and doesn't
  spin forever.
- **No secrets in images or logs.** Inject them at start, from a file outside the image, and keep
  them out of the agent's transcripts.

**Common mistakes.** "Please never push to main" in the instructions file, with nothing enforcing
it. Running in a mode that skips every permission check, outside a sandbox. Giving the agent a
personal token with full rights.

**In Pathfinder Dev.** Tasks run in containers; outward actions go through an action gate and are
refused unless granted (`--allow open-pr`, `--allow push`); the agent asks with `agentic-ask` and
waits for an answer; secrets come from `.agentic/.env` and never enter the image.

### Pillar 4 · The wiki

**What it is.** A small set of Markdown documents in the repository, written for the agent and read
before the code.

**Why it matters.** Without documentation written for the agent, the model falls back to its
defaults and reimplements what already exists. Of all nine pillars, this one surprised me most:
small wiki, big difference.

**How to build it.**

- **Markdown in the repo is the source of truth**, changed through pull requests. Other tools (a
  company wiki, for example) are for sharing, not for editing.
- **Document what the code can't tell:** the *why*, who owns what, decisions already made, how the
  business flows. **Skip what the agent can read itself:** structure, call chains, data model, API
  signatures.
- **Keep an index**, so the agent finds what it needs without reading everything.
- **Keep the decisions.** When you answer an agent's question or approve its work, the answer goes
  into the repo next to the spec. The next task reads it instead of asking again.
- **Separate three kinds of documentation:**

| | Agent wiki | Architecture docs | Business docs |
|---|---|---|---|
| **For** | The agent | Architects and the team | The business and the team |
| **Contains** | Intent, ADRs, component responsibilities, business flows | Full architecture (C4) | Processes and rules |
| **Size** | Light, indexed | Detailed | As needed |

**Common mistakes.** Giving the agent too much, or the same thing twice. Bloated or duplicated
documents crowd its context and dilute what matters. **Context management** is one of the main
skills of this whole setup.

**In Pathfinder Dev.** A wiki plugin gives the agent a routed project wiki; the `/architect` skill
keeps a light architecture description in `docs/architecture/` and reviews plans and code against
it.

### Pillar 5 · Instructions and skills

**What it is.** The layered instructions an agent loads: a project instructions file
(`CLAUDE.md`, `AGENTS.md`), rules for parts of the code, and skills for repeated tasks.

**Why it matters.** This is how you coach the agent once instead of in every prompt.

**How to build it.**

- **Always loaded:** the instructions file. The main concepts, the daily procedure, the commands,
  where to find what. Keep it short.
- **Loaded by path:** rules for one area, such as tests or the API layer.
- **Loaded on demand:** skills and templates for tasks that repeat.
- **Version and review it like code**, so an improvement made by one developer reaches the whole
  team.
- Add a **code index** (an LSP server or a code graph) so the agent spends fewer tokens searching.

**Common mistakes.** An instructions file of several thousand words. A skill for a task you do
twice a year: every skill is one more thing to maintain. Instructions that contradict the wiki.

**In Pathfinder Dev.** Skills, agents, commands and instructions in `.agentic/` are copied into
Claude Code and Copilot alike; `/agentic-setup` proposes the project's setup.

### Pillar 6 · Agentic colleagues

**What it is.** Agents with a standing job, running on a schedule instead of waiting for a ticket.

**Why it matters.** Some work is never urgent enough for a person to start. An agentic colleague
does it every time, without being asked.

**How to build it.**

- Give each colleague **one job**, a schedule and a clear output, usually a pull request or a
  report. Examples: a dependency updater at night, a tech-debt reviewer once a week, a
  documentation refresher after the day's merges, a nudger for unanswered customer questions.
- Add a **rule keeper**: once a week it reads the review comments on merged pull requests and
  proposes rule or wiki updates. You approve them like any other pull request. It turns "steer,
  don't fix" into a habit the system keeps for you.
- Tell a scheduled agent that **nobody is watching**, and give it the narrowest permissions that
  let it finish.
- Read the results. A colleague whose reports nobody reads is noise.

A real example: a project with a demanding client who talked to us on too many channels. The team
kept missing messages and the client kept complaining. We built a colleague with one job: find the
messages we missed and nudge the right person in chat.

**Common mistakes.** Colleagues with vague jobs ("improve the code"). Scheduled agents with
broad write access. Starting with colleagues before the guardrails are in place.

**In Pathfinder Dev.** `agentic schedule add nightly-deps --cron "0 2 * * 1-5" --allow open-pr
"..."` runs a task on a schedule, in a fresh container and worktree, and keeps a summary per run.

---

## Part 2: Your desk

### Pillar 7 · Spec-driven flow

**What it is.** A fixed flow, plan → implement → test, where you plan together with the agent and
the agent implements and tests on its own.

| Plan | Implement | Test |
|---|---|---|
| You and the agent | The agent | The agent |

**Why it matters.** The plan is the only step you are part of. Everything after it runs without
you, so the plan decides the result. **Invest in the plan: it is the new code.**

**How to build it.**

- **Adopt a known flow** instead of inventing one ([Superpowers](https://github.com/obra/superpowers)
  is a good start), then add your own quality gates: coverage, linters, security scans.
- **Plan together.** The agent asks questions and proposes an approach; you push back. Then step
  out.
- **Write the plan to a file** in the repo, so it survives the session and can be reviewed.
- Before you see the work, let **a second agent review it** against the plan and the rules. Your
  review then starts from a cleaner result.
- Know what a good plan contains:

| A good plan | A weak plan |
|---|---|
| Defines the architecture | Describes only the business need |
| Names the affected components and APIs | Leaves the agent to guess where code goes |
| States the reason and the business goal | |
| Sets test criteria and test cases | Says nothing about how to test |
| Says how the agent proves the work | |

A weak plan gets you *something*, rarely the thing you meant. A good plan gets you the thing you
would have built yourself.

**Common mistakes.** Pasting a ticket into the chat and hoping. Skipping the plan for "small"
changes that turn out not to be small. Reviewing the code instead of the plan.

**In Pathfinder Dev.** Superpowers is set up by default; Pathfinder Dev UI runs backlog
workflows such as "plan, implement, review" with a human gate after planning.

### Pillar 8 · Command center

**What it is.** One place where you start tasks, follow every agent, answer their questions and
review their results.

**Why it matters.** Once you stop watching sessions, the work needs a place to come back to you.
Without one you are tailing terminals, and that is babysitting, not managing.

**How to build it.** The command center needs four things:

- **Start a task** from a ticket or a sentence.
- **Follow every agent live**, local or remote, with its state and cost.
- **Answer its questions** when it stops and asks, and let it continue.
- **Review its pull request** together with the evidence it brings.

A command line works; a web UI scales better once you run more than a handful of tasks. Either
way, questions must reach you without you polling for them.

This pillar was the last one I added, after comparing my list with Hassan et al.,
[*Agentic Software Engineering*](https://arxiv.org/abs/2509.06216) (2025). They call it the
*Agent Command Environment*: the human's workbench, next to the agent's.

**Common mistakes.** One terminal tab per agent. Questions that wait for hours because nobody saw
them. No view of what a task has cost so far.

**In Pathfinder Dev.** From the CLI: `agentic run`, `follow`, `answer`, `status`, `cost`. In the
browser: Pathfinder Dev UI, with tasks, questions, backlog workflows and schedules.

### Pillar 9 · Proof of work

**What it is.** Evidence, delivered with every change, that the work does what it should.

**Why it matters.** If the agent can't prove its work, you become the full-time tester, and that is
Level 3. With the evidence in the pull request, review is reading, not re-testing. It is also the
bridge towards Level 5: the teams closest to it put most of their agents' effort into proving the
software works.

**How to build it.**

- **Unit tests with every change**, written with the code. Not optional.
- **The agent tests like you would:** with Playwright it clicks through the UI, with HTTP calls it
  exercises the API, against its own environment (pillar 2).
- **Regression suites** the agent writes, maintains and runs.
- **An evidence pack in every pull request:** what changed and why, the test results, screenshots,
  and, just as important, **what wasn't checked**.
- **Measure** time, cost and how often a task passes review without rework. Then you know whether a
  change to the setup really helped.

**Common mistakes.** Treating a green build as proof. Tests the agent wrote to pass, not to check.
Evidence packs that hide what was skipped.

**In Pathfinder Dev.** Playwright with Chromium is in the base image; tasks can start the app
inside their container; `agentic cost` reports what each task cost.

---

## Where to start

You don't need all nine on day one. This is the order I'd suggest, because each step pays off on
its own:

1. **Instructions and a small wiki (4, 5).** An afternoon of work, useful immediately, even at
   Level 2.
2. **Spec-driven flow and proof of work (7, 9).** Now you can hand over a task and get back
   something you can check. This is the step from Level 3 to Level 4.
3. **Toolbox container and guardrails (1, 3).** Now you can let the agent run without watching it.
4. **Command center (8).** Now you can run several tasks at once without losing track.
5. **Ephemeral environments (2).** Now parallel tasks don't step on each other.
6. **Agentic colleagues (6).** Last, because they need everything above to run safely alone.

Scale up step by step: start with one task; the next day, continue it and add a second in
parallel. The limit is your attention, not the agent.

## Limits and open questions

- **Cost.** Many agents running in parallel cost real money. Measure it per task from the start.
- **Ephemeral environments are the hardest pillar** for systems that depend on large shared
  databases or external partners. Mocks and seeded data help, but not everywhere.
- **Security.** Guardrails reduce risk; they don't remove it. Treat an agent like a new contractor
  with limited access.
- **The organisation.** When one team gets several times faster, the bottleneck moves to
  refinement, review, other teams and planning. That is the topic of the next article.

## Three habits

1. **Steer, don't fix by hand.** A manual fix teaches the agent nothing. A correction in a rule, the
   wiki or the plan template makes every later run better. This is literally why I haven't written
   a line of code in a year: every time I wanted to, I fixed the setup instead.
2. **Become the architect.** The agents write the code. You decide the structure, the boundaries
   and the constraints they work within.
3. **Master the business.** When code is cheap, knowing *what* to build is the scarce skill. You
   can't write a good spec for a business you don't understand.

## Try it

Everything above is implemented in **Pathfinder Dev**, which is open source:

- [Pathfinder Dev Base](https://github.com/popescualextraian/pathfinder-dev-base): the agent's
  container and CLI (toolbox, guardrails, skills, scheduled colleagues, detached tasks).
- [Pathfinder Dev UI](https://github.com/popescualextraian/pathfinder-dev-ui): the command center.

Try it, and tell me what breaks. The next articles in this series go deeper, one pillar at a time.

## References

- Dan Shapiro, "The Five Levels", January 2026.
- Ahmed E. Hassan et al., [*Agentic Software Engineering: Foundational Pillars and Research
  Directions*](https://arxiv.org/abs/2509.06216), arXiv:2509.06216, 2025.
- [Superpowers](https://github.com/obra/superpowers), a skills library for coding agents with a
  plan, implement and review flow.
