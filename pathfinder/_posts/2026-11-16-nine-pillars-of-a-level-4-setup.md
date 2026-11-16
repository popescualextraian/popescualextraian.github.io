---
layout: post
title: "I haven't written a line of code in over a year. Here's the setup."
description: "The nine pillars of a Level 4 agentic development setup, from my IT Days 2026 talk."
---

I haven't written a line of code in over a year.

No, I didn't quit development and become a manager. Well… half true. I manage agents now. I
architect, I delegate, I coach them, I review their work. My colleagues are agents.

This post is the written version of my talk at IT Days 2026 in Cluj. It's the first in a series
on **Pathfinder**: the ideas behind a Level 4 setup, and the open-source tools that implement them.

## Your new role

| Before | Now |
|---|---|
| Write the code | Architect the solution |
| Fix the bug | Coach the agent |
| Do the task | Delegate the task |
| Review my own work | Review my colleagues' work |

## The five levels

Dan Shapiro describes five levels of AI-assisted development, from spicy autocomplete to the dark
factory ("The Five Levels", January 2026):

| | Level | The human |
|---|---|---|
| 0 | Spicy autocomplete | Writes everything; AI suggests the next line |
| 1 | The coding intern | Writes the important code; AI writes boilerplate |
| 2 | The junior developer | Pair programs; reviews every line |
| 3 | The developer | Full-time code reviewer |
| 4 | The engineering team | Writes specs, manages agents |
| 5 | The dark factory | Not in the loop |

Most developers who call themselves AI-native stop at Level 2. Level 2 feels done. It isn't.

Level 4 is where I work every day. Level 5? Not achievable yet, in my opinion. The few teams there
are tiny and spend most of their effort proving the software works.

So how do you get to Level 4? Not with a smarter model. The models are good enough already. You
need the setup around them. Which raises the real question:

**What does the architecture of such a system look like?**

## Nine pillars, two groups

I break it into nine pillars:

- **The agent's workplace**, where it works and what it knows:
  1 Toolbox container · 2 Ephemeral environments · 3 Guardrails · 4 The Wiki ·
  5 Instructions and skills · 6 Agentic colleagues
- **Your desk**, what you need to manage the agents:
  7 Spec-driven flow · 8 Command center · 9 Proof of work

Those last three are your whole job at Level 4.

## The agent's workplace

### 1 · Toolbox container: one container with every tool, on a laptop or a VM

git, the coding agents (Claude Code, Copilot CLI), Java, Node, Python, Docker and cloud CLIs, plus
each project's own build tools, linters, Playwright, database clients and MCP servers. The same
container runs on your laptop (a few agents, only while it's on) or on a remote VM (as many as it
holds, all night).

**Why it matters.** An agent can only do a developer's work with a developer's tools. Without them
it guesses. If a task needs a tool, put it in the image; don't work around it. Start locally; if
nothing depends on the host, moving to a VM is a deployment, not a rewrite.

And running remotely changes how you behave. When the agent runs on your laptop, you keep watching
the session. When it runs on a VM, you start it and stop looking. **Out of sight is what makes you a
manager.** You stop babysitting and start delegating.

### 2 · Ephemeral environments: every agent gets its own environment

Each branch deploys to its own isolated environment: the app, its database, its dependencies.
Started on demand, removed when the work is merged.

**Why it matters.** Each agent must test its work independently, in parallel with the others. One
shared dev environment means agents overwrite each other's changes, or wait in line. Five agents on
five stories need five environments. Otherwise you're back to one at a time.

### 3 · Guardrails, not supervision

- **The container is the sandbox:** the agent can break its own box, not your machine.
- **Hard rules live in code, not in prompts:** enforced by scripts and hooks, not by asking nicely.
- **A gate on outward actions:** pushing, posting and deploying pass a check first.
- **Branch and pull request only:** never straight to main.
- **Clear rules for when it stops and asks:** a rule says so, it's unsure, a check keeps failing,
  or the budget runs out.
- **No secrets in images or logs.**

**Why it matters.** You can't walk away from something you have to watch. Guardrails replace
supervision, and that's what makes remote agents safe. I told you I stop looking at the session.
This is why I can.

### 4 · The Wiki: a wiki the agent reads before the code

Markdown in the repo is the source of truth, and changes go through pull requests. Document what
the code can't tell: the *why*, who owns what, decisions already made, how the business flows.
Skip what the agent can read itself: structure, call chains, data model, API.

I like three kinds of documentation:

| | Agent wiki | Architecture docs | Business docs |
|---|---|---|---|
| **For** | The agent | Architects and the team | The business and the team |
| **Contains** | Intent, ADRs, component responsibilities, business flows | Full C4 architecture | Business processes and rules |
| **Size** | Light, indexed | Detailed | As needed |

**Why it matters.** Without docs written for the agent, the model falls back to its defaults and
reimplements what already exists. But don't give it too much, or the same thing twice: bloated docs
crowd its context and dilute what matters. **Context management** is one of the key things that make
this whole setup work.

One more habit: keep the decisions. When you answer the agent's question or approve its work, that
answer goes into the repo next to the spec. The next task reads it instead of asking again.

The agent wiki is the one that surprised me most. Small wiki, big difference.

### 5 · Instructions and skills: CLAUDE.md, rules and skills

- **Always loaded:** CLAUDE.md. Main concepts, daily procedure, where to find what.
- **Loaded by path:** rules for one area, such as tests or the API layer.
- **Loaded on demand:** skills and templates for repeated tasks.

**Why it matters.** This is how you coach the agent once instead of in every prompt. Keep it light:
every skill is one more thing to maintain. Same lesson as the wiki: load what's needed, when it's
needed. Tip: a code index (an LSP server, for example) saves the tokens the agent would spend
searching.

### 6 · Agentic colleagues: colleagues with a standing job

Not every agent waits for a ticket. Some have one job and do it on their own schedule: a nudger
for unanswered customer questions, a tech-debt reviewer, a documentation refresher after each day's
merges, and **the rule keeper**, which reads the review comments on merged pull requests and
proposes rule or wiki updates.

**Why it matters.** Some work is never urgent enough for a person to start. An agentic colleague
does it every time, without being asked.

A real one: a project with a very demanding client, talking to us on too many channels. Nobody
could keep track, the team kept forgetting to answer, and the client kept complaining. So we built
an agentic colleague with one job: find the messages we missed and nudge us directly in chat.

## Your desk

### 7 · Spec-driven flow: plan → implement → test

| Plan | Implement | Test |
|---|---|---|
| You and the agent | The agent | The agent |

Adopt a known flow (Superpowers, for example) and add your own checks: quality gates, coverage,
security scans.

**Why it matters.** The plan is the only step you're part of. Everything after it runs without
you, so the plan decides the result. **Invest in the plan: it is the new code.**

| A good plan | A weak plan |
|---|---|
| Defines the architecture | Describes only the business need |
| Names the affected components and APIs | Leaves the agent to guess where code goes |
| States the reason and the business goal | |
| Sets test criteria and test cases | Says nothing about how to test |
| Says how the agent proves the work | |

A weak plan gets you *something*, rarely the thing you meant. A good plan gets you the thing you
would have built yourself.

### 8 · Command center: one place to manage your agents

- **Start a task** from a ticket or a sentence
- **Follow every agent** live, local or remote
- **Answer its questions** when it stops and asks
- **Review its pull request** and the evidence it brings

**Why it matters.** Once you stop watching the session, the work needs a place to come back to
you. Without one you're tailing terminals. That's babysitting, not managing.

This pillar came last. I added it after comparing my list with Hassan et al.,
[*Agentic Software Engineering*](https://arxiv.org/abs/2509.06216) (2025), which calls it the
*Agent Command Environment*: the human's workbench next to the agent's.

### 9 · Proof of work: evidence, not a green build

Unit tests with every change. Playwright to click through the UI like you would. Regression suites
the agent writes and runs. **An evidence pack in every pull request:** what changed and why, the
results, screenshots, and what wasn't checked.

**Why it matters.** If the agent can't prove its work, you become the full-time tester, and that's
Level 3. With the evidence in the pull request, review is reading, not re-testing. It's also the
bridge to Level 5: the dark-factory teams put most of their agents' effort into proving the
software works.

## Three habits to take home

1. **Steer, don't fix by hand.** A manual fix teaches the agent nothing. A correction in the
   prompt, a rule or the wiki makes every later run better. This is literally why I haven't written
   a line of code in a year: every time I wanted to, I fixed the setup instead.
2. **Become the architect.** The agents write the code. You decide the structure, the boundaries
   and the constraints they work within.
3. **Master the business.** When code is cheap, knowing *what* to build is the scarce skill. You
   can't write a good spec for a business you don't understand.

## Try it

Everything above is implemented in **Pathfinder Dev**, and it's open source:

- [Pathfinder Dev Base](https://github.com/popescualextraian/pathfinder-dev-base): the agent's
  container and CLI (toolbox, guardrails, skills, scheduled colleagues).
- [Pathfinder Dev UI](https://github.com/popescualextraian/pathfinder-dev-ui): the command center
  (tasks, questions, workflows, schedules).

Try it, star it, and tell me what breaks. Next in this series: how each pillar is built in
Pathfinder Dev.
