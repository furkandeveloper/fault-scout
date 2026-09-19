---
description: Discover how this repository's system could fail (FaultScout chaos analysis)
argument-hint: "[language=english|turkish]"
---

Run a FaultScout chaos analysis on the current repository.

Load and follow the `chaos-analysis` skill for the full rules. This command is
**analysis-only**: read the repository, reason about it, and print a report.
Do not create or modify any file in the repository, do not change
configuration, and do not start, stop, or disrupt any process, container,
network, or service.

## Usage

```text
/faultscout:chaos                     report in English (default)
/faultscout:chaos language=english    report in English
/faultscout:chaos language=turkish    report in Turkish
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

The language applies only to the human-readable report in step 10. The
analysis itself (steps 1–9) is unaffected. The "Report language" section of
the skill defines what is translated and what must never be translated: file
paths, function/type/variable names, config keys, queue/exchange/topic names,
database and Redis key names, log messages, and code snippets are always
quoted exactly as they appear in the repository, in every language.

## Steps

Work through these steps in order. Steps 1–8 are silent analysis: use tools,
take normalized notes, and at most print one-line progress remarks. Do not
print drafts, candidate lists, or partial scenarios. The report is printed
once, in step 10, as the final message.

1. **Inspect the repository.** Identify languages, entry points, dependency
   manifests, configuration, Docker/infrastructure files, database access,
   caches, message brokers, event producers/consumers, background workers,
   scheduled jobs, external API calls, tests, retries, timeouts, and
   transactions. Only report what the repository actually contains.
2. **Understand the architecture.** Build a short mental model of the main
   components and how requests, data, and events move between them. This
   model becomes the Mermaid architecture diagram in the report, so keep
   track of which file proves each component and each connection.
3. **Identify dependencies.** List the internal and external systems the code
   depends on, with the files that prove it.
4. **Trace the important flows.** Follow the most business-critical paths end
   to end (e.g. request → handler → database → event → consumer). Read the
   actual code on those paths; do not infer behavior from file names.
5. **Identify failure boundaries.** Find the points where a network call can
   fail, an operation can partially complete, a process can crash between two
   side effects, an event can be duplicated/delayed/reordered, retries can
   repeat side effects, data can be stale, or concurrent operations can race.
6. **Generate candidate failure hypotheses.** For each boundary, write
   Condition + Failure + Affected component + Potential consequence + Root
   cause, tied to concrete evidence. Produce more than five candidates.
7. **Deduplicate by root cause.** Merge candidates that share a root cause and
   propagation path into one stronger scenario. Keep candidates separate only
   when the propagation path or affected component genuinely differs.
8. **Select the five strongest scenarios.** Rank by evidence strength,
   realism, impact, clarity of propagation, and future testability. Cover
   different failure classes where the repository supports it; never invent a
   scenario to fill a class. If fewer than five are well evidenced, report
   fewer and say so.
9. **Self-check the draft.** Run the pre-output self-check from the skill:
   fixed structure, every section exactly once per scenario, no duplicated
   evidence or paragraphs, no near-duplicate titles, no merged or truncated
   words, no raw tool output, every path and symbol actually seen, hedged
   language only, every diagram consistent with its text (no invented nodes
   or edges), and — when the report is not in English — the language checks
   (headings from the skill's table, identifiers untranslated, hedging
   preserved, Mermaid syntax intact). Fix problems before printing.
10. **Print the FaultScout Report** once, in the chat, in the resolved report
    language, using the exact report and scenario format defined in the
    skill: a Mermaid architecture diagram of the normal flow under
    Architecture, the Failure Propagation text chain for every scenario, and
    a Mermaid propagation diagram only for scenarios where it adds value (a
    branch, a join, a retry loop, or a long chain). Keep it concise.

## Rules that always apply

- Evidence before hypothesis. Never invent files, functions, services,
  dependencies, configuration, behavior, or line numbers. Cite line numbers
  only for lines you actually read.
- If evidence for a scenario cannot be found, write `Evidence insufficient.`
  (or its translation from the skill's heading table).
- Never claim the system definitely fails. Use "may", "could", "potential",
  "plausible", or "likely" (only when evidence supports it), or their
  equivalents in the report language. No result in this report is confirmed.
- Confidence is High, Medium, or Low and measures confidence in the analysis,
  not severity. Translating the report never changes a confidence level.
- The report is written in clean prose from your own notes, never by pasting
  tool output. Each scenario heading and section appears exactly once.
- Diagrams are plain `mermaid` fences and follow the same evidence rule as
  text: no component, connection, or propagation step without repository
  evidence, and no wording that claims a confirmed failure.
- Code identifiers are never translated, in any language.
