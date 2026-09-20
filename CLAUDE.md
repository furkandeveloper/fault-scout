# FaultScout — CLAUDE.md

> **Note: this file is a development specification, not plugin runtime
> context.** Claude Code does not load a plugin's root `CLAUDE.md` when the
> plugin is installed. Everything FaultScout sends to Claude at runtime comes
> from `commands/analyze.md` and `skills/faultscout/SKILL.md`. This file only
> guides Claude while *developing* FaultScout in this repository. (Plugin
> validation may report it as a warning for the same reason; that is
> expected.)

## Project Overview

**FaultScout** is an open-source Claude Code plugin that helps developers
discover how their software systems could fail.

> **FaultScout: Discover how your software system can fail.**

It is installed into a developer's existing Claude Code environment and used
inside their own repositories. Its only capability is **evidence-based static
reliability analysis** of a repository.

FaultScout is not a generic AI chatbot and not a chaos-experiment runner. It
is a specialized reliability-analysis skill whose core question is:

> Given this repository's actual architecture and implementation, how could
> this system fail under abnormal conditions?

---

# Scope

FaultScout is **analysis-only**. It reads a repository and prints a report.

It must NOT:

- modify the analyzed repository's source code, configuration, or files
- run the analyzed project, its tests, or its build
- execute runtime experiments or failure injection of any kind
- manipulate containers, networks, processes, caches, databases, or brokers
- use Docker, Docker Compose, Kubernetes, or cloud CLIs
- access production or any external system
- present a scenario as tested, reproduced, observed, or confirmed
- invent evidence, components, behavior, line numbers, or runtime results

Do NOT implement, now or as a "future phase" inside this repository:

- MCP servers or MCP configuration
- backend services, daemons, databases, web UIs, dashboards
- snapshot / restore logic, experiment approval flows, experiment result
  enums
- automatic code changes or automatic fixes
- external dependencies, build steps, or runtime code of any language

The whole product is Claude Code instructions:

```text
Repository
    ↓
Read-only discovery
    ↓
Architecture + critical flows
    ↓
Failure boundaries + existing safeguards
    ↓
Evidence-backed failure scenarios
    ↓
FaultScout Analysis Report (Markdown, printed in chat)
```

---

# Architecture

FaultScout is a **Claude Code plugin**. Do not build a separate application.

```text
Claude Code
    │
    ├── FaultScout plugin
    │
    ├── /faultscout:analyze command   (commands/analyze.md)
    │
    └── faultscout skill              (skills/faultscout/SKILL.md)
            │
            ▼
      Repository discovery
            │
            ▼
      Architecture and critical flows
            │
            ▼
      Failure scenarios with evidence
            │
            ▼
      FaultScout Analysis Report
```

Claude Code already provides file inspection, search, Git, and shell access.
Do not recreate these capabilities.

---

# Repository Structure

Keep the project intentionally small:

```text
fault-scout/
├── .claude-plugin/
│   ├── plugin.json          plugin manifest
│   └── marketplace.json     marketplace manifest (installs from GitHub)
├── commands/
│   └── analyze.md           /faultscout:analyze
├── skills/
│   └── faultscout/
│       └── SKILL.md         the analysis skill
├── CLAUDE.md                this development specification
├── README.md
└── LICENSE
```

Do not add directories such as `src/`, `analyzer/`, `services/`, `core/`,
`utils/`, or any code tree. Do not add a second command or skill unless
there is a concrete, analysis-only requirement, and explain the deviation.

The primary skill is named **`faultscout`**. The command is
**`/faultscout:analyze`**. Do not reintroduce the names `chaos-analysis`,
`chaos-experiment`, `/faultscout:chaos`, or `/faultscout:experiment`.

---

# Plugin Manifest

`.claude-plugin/plugin.json` uses the Claude Code plugin manifest format:
`name` (`faultscout`), `description`, `version`, `author`, `license`,
`keywords`, `commands: "./commands/"`, `skills: "./skills/"`. Keep
`.claude-plugin/marketplace.json` in sync (same description and version).

Do not add hooks, MCP servers, agents, or other configuration.

---

# `/faultscout:analyze` Command

`commands/analyze.md` invokes the `faultscout` skill and performs read-only
repository analysis. It:

1. Resolves the report language from `$ARGUMENTS` (`language=english` by
   default, `language=turkish` supported; any other value prints an
   "Unsupported language" message and stops before analysis).
2. Walks the analysis steps silently: discovery → architecture and critical
   flows → trace the flows in code → failure boundaries → existing
   safeguards → candidate scenarios → ranking and confidence → self-check.
3. Prints the FaultScout Analysis Report once, as the final message.

The command must never modify the analyzed repository and must not execute
runtime experiments.

---

# `faultscout` Skill

`skills/faultscout/SKILL.md` defines Claude's role as a senior reliability
engineer performing static analysis. Its rules, in priority order:

## Evidence before hypothesis

Every claim rests on something actually found in the repository in the
current session. Never invent files, functions, classes, services,
dependencies, databases, brokers, configuration, behavior, or line numbers.
Line ranges are cited only for lines actually read.

Two symmetric rules: a missing safeguard is a gap, not a proven bug; an
unobserved failure is not an impossible one.

## Three levels of certainty

| Level | Phrase |
|---|---|
| Directly observed | "The code directly shows..." |
| Strongly supported inference | "This suggests..." |
| Plausible but unverified | "A possible failure mode is..." / "The repository does not provide enough evidence to confirm..." |

Never state that the system definitely fails; use may / could / potential /
plausible / likely (only when evidence supports it).

## Discovery

Languages, services, modules, entry points, dependency manifests,
configuration, infrastructure and CI/CD files, databases and transactions,
caches, brokers and producers/consumers, workers and scheduled jobs,
external APIs, retries / timeouts / circuit breakers, health checks and
shutdown, resource limits, tests. Only report what the repository contains.

## Failure classes

Dependency outages, timeouts, retries and retry storms, missing resilience
safeguards, partial writes, stale cache, race conditions, duplicate
processing, event delivery failures, idempotency gaps, startup failures,
resource exhaustion, recovery problems. Never invent a scenario to fill a
class.

## Scenario selection

Generate candidates internally, deduplicate by root cause, rank by evidence
strength, realism, impact, and clarity of propagation. Report the strongest
(typically three to seven), covering different classes where supported;
report fewer and say so rather than pad.

## Confidence

`HIGH` (directly supported by clear code or configuration evidence),
`MEDIUM` (strongly plausible, multiple pieces of evidence, runtime unknown),
`LOW` (possible, evidence insufficient). Confidence describes the quality of
the static evidence, never probability or severity.

## Scenario fields

Title, Summary, Trigger condition, Affected component, Evidence (file
paths; function / method / class / configuration references; line ranges
when available), Failure mechanism, Failure propagation path, Potential
consequences, Existing mitigations, Missing or questionable mitigations,
Static confidence, Suggested follow-up, Evidence limitations. Each exactly
once, in that order.

## Report sections

```text
# FaultScout Analysis Report
## 1. Executive Summary
## 2. Repository Overview
## 3. Architecture and Critical Flows
## 4. Failure Scenarios
## 5. Failure Propagation Paths
## 6. Existing Resilience Mechanisms
## 7. Potential Reliability Gaps
## 8. Recommended Follow-Up Actions
## 9. Evidence and Limitations
```

Fixed order and headings. Mermaid diagrams (service dependency graph,
request lifecycle, data consistency flow, event processing flow, failure
propagation flow) only where they clarify an evidence-supported
relationship; every node and edge backed by a file that was read; never
decorative.

## Report language

English by default, Turkish supported. Only human-readable text is
translated; identifiers, paths, config keys, names taken from the
repository, and Mermaid syntax are never translated. Translation never
changes evidence, certainty, hedging, confidence, or diagram structure. The
skill holds the Turkish heading table; adding a language means adding a
table there and the value to the supported list in the skill and the
command.

---

# Development Instructions for Claude

When changing FaultScout:

1. Keep it analysis-only. If a requested feature would run, mutate, or
   observe a live system, explain that it is out of scope and leave it out.
2. Keep the structure to the files listed above. Explain any deviation.
3. Keep the command and the skill consistent with each other: same steps,
   same section and field names, same supported languages, same boundaries.
4. Keep the skill free of ambiguous, conflicting, or overly broad
   instructions. Every rule should be checkable in the pre-output
   self-check.
5. Keep README, CLAUDE.md, and both manifests describing the same product
   and the same version.
6. After any change, run:

   ```bash
   claude plugin validate .
   grep -rniE "mcp|experiment|inject|snapshot|restore|chaos-analysis|chaos-experiment|faultscout:chaos" --include='*.md' --include='*.json' .
   ```

   Validation must pass, and the grep must return only intentional
   mentions (for example this file's list of things not to build, or the
   README's statement that FaultScout does not inject failures).
7. Test with a local checkout: `claude --plugin-dir /path/to/fault-scout`
   inside some other repository, then `/faultscout:analyze`.

The goal is not to build a large codebase. The goal is to prove that:

> **Claude Code can use FaultScout to look at a real repository and discover
> meaningful, evidence-backed ways that system could fail — without running
> or touching it.**
