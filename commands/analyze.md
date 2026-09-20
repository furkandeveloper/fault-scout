---
description: Read-only reliability analysis of this repository — discover how the system could fail (FaultScout)
argument-hint: "[language=english|turkish]"
---

Run a FaultScout analysis on the current repository.

Load and follow the `faultscout` skill for the full rules. This command is
**analysis-only**: read the repository, reason about it, and print a
FaultScout Analysis Report in the chat. Never create, modify, or delete a
file in the repository as part of the analysis (write the report to a file
only if the user explicitly asks for one and names the destination); never
run the project, its tests, or its build; never start,
stop, or disrupt a process, container, network, or service; never contact
an external system; never perform a runtime experiment or present a
scenario as tested, reproduced, or confirmed.

## Usage

```text
/faultscout:analyze                     report in English (default)
/faultscout:analyze language=english    report in English
/faultscout:analyze language=turkish    report in Turkish
```

## Report language

Arguments passed to this command: `$ARGUMENTS`

Resolve the report language **before** starting the analysis:

- No arguments, or no `language=` option → `english`.
- `language=<value>` → use `<value>`, compared case-insensitively.
- Supported values are exactly `english` and `turkish`.
- Anything in the arguments that is not a `language=` option is ignored.

If the value is not supported, print exactly this (with the value the user
gave) and **stop**. Do not inspect the repository and do not produce a report:

```text
Unsupported language: <value>.
Supported languages: english, turkish.
```

The language applies only to the human-readable report in step 9. The
analysis itself is unaffected. The "Report language" section of the skill
defines what is translated and what is never translated (file paths,
symbols, configuration keys, queue/table/key names, log messages, code
snippets, and Mermaid syntax are always quoted exactly as they appear in
the repository).

## Steps

Work through these steps in order. Steps 1–8 are silent analysis: use
read-only tools, take normalized notes, and print at most one-line progress
remarks. Do not print drafts, candidate lists, or partial scenarios. The
report is printed once, in step 9, as the final message.

1. **Discover the repository.** Identify languages, frameworks, services and
   modules, entry points, dependency manifests, configuration, databases,
   caches, message brokers, external APIs, background workers, scheduled
   jobs, infrastructure files, and CI/CD files. Only note what the
   repository actually contains.
2. **Reconstruct the architecture and critical flows.** Build a model of
   the components and the connections you traced, and pick the two to five
   flows that matter most. Track which file proves each component and edge.
3. **Trace the critical flows in the code.** Read the handlers, clients,
   consumers, jobs, and data access on those paths. Do not infer behavior
   from file or function names.
4. **Locate failure boundaries.** Note every point where a dependency can
   be unavailable or slow, a timeout is missing or unbounded, a retry can
   repeat a side effect, a write can partially complete, a cache can go
   stale, concurrent operations can race, an event can be duplicated,
   delayed, lost, or reordered, startup can fail, a resource can be
   exhausted, or recovery is unclear.
5. **Catalogue existing resilience mechanisms.** Retries, timeouts,
   circuit breakers, idempotency keys, transactions, outboxes, health
   checks, graceful shutdown, dead-letter handling — with the files that
   prove them.
6. **Generate candidate failure scenarios**, more than you will report, each
   tied to concrete evidence and a root cause. Merge candidates that share
   a root cause and propagation path.
7. **Select and rank the strongest scenarios** by evidence strength,
   realism, impact, and clarity of propagation, covering different failure
   classes where the repository supports it. Assign HIGH / MEDIUM / LOW
   static confidence to each, based on evidence quality only. Never pad;
   if the evidence supports fewer scenarios, report fewer and say so.
8. **Self-check the draft** against the skill's pre-output self-check:
   fixed nine-section structure, every scenario field exactly once, every
   path and symbol actually seen, every line range actually read, certainty
   phrases on every claim, hedged language only, no claim of a tested or
   confirmed result, diagrams only where they clarify evidence and
   consistent with their text, and — when the report is not in English —
   the language checks.
9. **Print the FaultScout Analysis Report** once, in the chat, in the
   resolved language, using the exact report and scenario format defined in
   the skill.

## Rules that always apply

- Evidence before hypothesis. Never invent files, functions, services,
  dependencies, configuration, behavior, or line numbers.
- Distinguish directly observed behavior ("The code directly shows..."),
  strongly supported inference ("This suggests..."), and plausible but
  unverified hypotheses ("A possible failure mode is...", "The repository
  does not provide enough evidence to confirm..."). Never state that the
  system definitely fails.
- A missing safeguard is a gap, not a proven bug; an unobserved failure is
  not an impossible one.
- Confidence describes the quality of the static evidence, never the
  probability or severity of the failure.
- The report is written in clean prose from your own notes, never by
  pasting tool output.
- Diagrams follow the same evidence rule as text and appear only where they
  clarify an evidence-supported relationship.
- Code identifiers are never translated, in any language.
