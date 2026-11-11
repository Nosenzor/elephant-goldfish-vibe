# Elephant-Goldfish Model (EGM) for Mistral Vibe

**Design with an elephant. Review with a goldfish.**

This is a standalone [Agent Plugins 1.0](https://agent-plugins.org) package that
ports the [Elephant-Goldfish Model](https://github.com/google/elephant-goldfish-model)
(Google, Apache-2.0) to Mistral Vibe. It adds spec-driven development skills to
Vibe: the **Elephant** is your main session (full project context) which explores
ideas and drafts specifications; **Goldfish** are fresh-context subagents that
review documents without the earlier conversation, exposing unstated assumptions,
hallucinations, and blind spots. You make the trade-offs and approve the spec
before implementing.

## Install

Vibe discovers plugins in `~/.vibe/plugins/<name>/` (user scope — every project)
and `.vibe/plugins/<name>/` (project scope — trusted folders only).

**One-liner (user scope):**

```sh
git clone https://github.com/Nosenzor/elephant-goldfish-vibe.git ~/.vibe/plugins/egm
```

Then run `/reload` in Vibe (or start a new session). The skills appear as
`/egm:*` slash commands and the reviewer agents as `egm:<persona>` subagents.
Check what loaded with `/plugins`.

**Alternatives:**

```sh
# Project scope: share the plugin with everyone who clones and trusts the repo
git clone https://github.com/Nosenzor/elephant-goldfish-vibe.git .vibe/plugins/egm

# Manual: download this repo as a zip and unpack it into either plugins dir
# (the unpacked folder must be named "egm" and contain plugin.json at its root)
```

**Update:** `git -C ~/.vibe/plugins/egm pull` then `/reload-plugins` in Vibe.

## Commands

| Command | Use it to |
| --- | --- |
| `/egm:brainstorm <idea>` | Test an idea with a 7-lens Goldfish shootdown; writes `concepts_brief.md` |
| `/egm:draft` | Turn the brief into a section-by-section design doc (`design_draft.md` → `docs/designs/`) |
| `/egm:review <file>` | Five-stage fresh-context review of a design document |
| `/egm:review-loop` | Configurable iterative review loop (converges on 2 clean rounds, cap 5) |
| `/egm:resolve` | Triage findings (CONTEST / ESCALATE / PROPOSE FIX / DECLINE) and apply fixes |
| `/egm:bootstrap <path>` | Generate EGM-compliant README contracts across a codebase subtree |
| `/egm:drift-report <dir>` | Compare checked-in docs against the current source code |

You can use each skill on its own; you do not have to follow the full sequence.

## How Goldfish run on Vibe

Each reviewer persona is a plugin subagent in `ai.mistral.vibe/agents/*.toml`
(namespaced `egm:<name>`, e.g. `egm:zoom-out`, `egm:design-critic`,
`egm:comprehension`, `egm:structural`, `egm:style`,
`egm:implementation-readiness`, `egm:codebase-details`,
`egm:inventive-catalyst`, `egm:synthesizer`). Skills spawn them with the `task`
tool (`agent: "egm:zoom-out"`), so every review runs in a fresh context window —
that isolation is what makes it a Goldfish review. Reviewers are read-only
(`safety = "safe"`, only `read_file` / `grep` / `web_fetch`); they return their
report as the task result. The skills dispatch reviewers in parallel (flat
swarm) and aggregate the findings in the main session.

### Reviewing with a different model (the duck)

By default reviewers run on the session's active model. To have a different
model review — the same one you use for `/peer` rubber-duck review — set
`active_model` in the reviewer TOMLs to your duck's model alias:

```toml
# ai.mistral.vibe/agents/zoom-out.toml (and siblings)
active_model = "my-duck"    # the same alias you configured under [utility_models] peer
```

Then `/reload`. The `task` tool has no per-spawn model override, so the agent
profile is the place to pin it.

## Conventions

- **Artifact-first**: drafts and reports are written to files in the working
  directory (`concepts_brief.md`, `design_draft.md`, `*_report.md`,
  `docs/designs/<feature>.md`); chat only gets the path. Never dump long
  documents into chat.
- **Gates**: interactive checkpoints use `ask_user_question` — strictly one
  question per call, as the sole tool call in the turn, then yield. Open-ended
  input (file paths) is asked in plain chat.
- **Rubrics**: the document standards reviewers audit against live in
  `resources/standards/` (feature-spec, implementation-readiness, zoom-out,
  heilmeier, re-implementation); shared behavior rules in
  `resources/execution-rules.md`.
- Severities are text: `Blocker | Major | Nit`.

## Attribution & license

Ported from [google/elephant-goldfish-model](https://github.com/google/elephant-goldfish-model)
(Apache-2.0). Original essay: Dave Rensin, *Elephants, Goldfish, and the New
Golden Age of Software Engineering*. See `NOTICE` and `LICENSE`.
