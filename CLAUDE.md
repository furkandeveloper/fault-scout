# FaultScout — CLAUDE.md

> **Note: this file is a development specification, not plugin runtime
> context.** Claude Code does not load a plugin's root `CLAUDE.md` when the
> plugin is installed. Everything FaultScout sends to Claude at runtime comes
> from `commands/chaos.md` and `skills/chaos-analysis/SKILL.md`. This file only
> guides Claude while *developing* FaultScout in this repository. (Plugin
> validation reports it as a warning for the same reason; that is expected.)

## Project Overview

Build **FaultScout**, an open-source Claude Code plugin that helps developers discover how their software systems could fail.

> **FaultScout: Discover how your software system can fail.**

FaultScout is intended to be installed into a developer's existing Claude Code environment and used inside their own repositories.

The first version must be extremely small.

---

# Core Idea

The user should be able to open Claude Code inside any software repository and run:

```text
/chaos
```

FaultScout should then:

1. Inspect the repository.
2. Understand the system architecture.
3. Identify important dependencies and flows.
4. Identify potential failure boundaries.
5. Generate concrete failure hypotheses.
6. Produce a report containing the **5 strongest potential failure scenarios**.

The scenarios must be based on evidence from the actual repository.

The system must NOT produce generic chaos-engineering advice.

---

# Important Product Principle

FaultScout is not a generic AI chatbot.

It is a **specialized reliability/chaos-engineering skill for Claude Code**.

Its core question is:

> "Given this repository's actual architecture and implementation, how could this system fail under abnormal conditions?"

---

# Phase 0.1 Scope

Phase 0.1 is **analysis-only**.

The plugin must NOT actually disrupt or modify the user's system.

Do NOT implement:

* Python
* Backend server
* Database
* MCP server
* Docker integration
* Kubernetes integration
* Network fault injection
* Process killing
* Service shutdown
* Redis manipulation
* PostgreSQL manipulation
* Kafka manipulation
* Elasticsearch manipulation
* Production infrastructure integration
* Automatic code changes
* Automatic fixes
* Web UI
* Dashboard
* Vector database
* RAG infrastructure
* Model training

These are future possibilities.

The only goal of Phase 0.1 is:

```text
Repository
    ↓
Analysis
    ↓
Failure hypotheses
    ↓
5 concrete scenarios
    ↓
Markdown report
```

---

# Architecture

FaultScout must be implemented as a **Claude Code Plugin**.

Do not build a separate application.

The initial architecture should be:

```text
Claude Code
    │
    ├── FaultScout Plugin
    │
    ├── /chaos command
    │
    └── chaos-analysis skill
            │
            ▼
      Repository Analysis
            │
            ▼
      Failure Hypotheses
            │
            ▼
       Chaos Report
```

Claude Code already provides the ability to inspect files, search the repository, use Git, and execute commands.

Do not recreate these capabilities.

---

# Initial Repository Structure

Keep the project intentionally small.

Use this structure:

```text
faultscout/
│
├── .claude-plugin/
│   └── plugin.json
│
├── skills/
│   └── chaos-analysis/
│       └── SKILL.md
│
├── commands/
│   └── chaos.md
│
├── README.md
└── LICENSE
```

Do not create additional directories unless there is a concrete requirement.

Do not create:

```text
src/
analyzer/
services/
models/
repositories/
clients/
utils/
core/
domain/
application/
infrastructure/
```

There is no need for them in Phase 0.1.

---

# Plugin Manifest

Create:

```text
.claude-plugin/plugin.json
```

Use the Claude Code plugin manifest format.

Initial metadata should be approximately:

```json
{
  "name": "faultscout",
  "description": "Discover how your software system can fail.",
  "version": "0.1.0",
  "author": {
    "name": "FaultScout"
  },
  "license": "MIT",
  "keywords": [
    "chaos-engineering",
    "reliability",
    "claude-code",
    "agent",
    "software-engineering"
  ],
  "skills": "./skills/"
}
```

Do not add unnecessary configuration.

---

# `/chaos` Command

Create:

```text
commands/chaos.md
```

The command must invoke the chaos-analysis workflow.

The user experience should be:

```text
/chaos
```

Claude then analyzes the current repository.

The command should instruct Claude to:

1. Inspect the repository.
2. Understand the architecture.
3. Identify dependencies.
4. Trace important flows.
5. Identify failure boundaries.
6. Generate failure hypotheses.
7. Select the five strongest scenarios.
8. Produce a concise Chaos Report.

---

# Chaos Analysis Skill

Create:

```text
skills/chaos-analysis/SKILL.md
```

The skill should define Claude's role as a senior reliability/chaos engineer.

The skill must emphasize:

> Evidence before hypothesis.

Claude must never invent:

* files
* functions
* services
* dependencies
* databases
* message brokers
* architecture
* behavior
* configuration
* line numbers

If evidence cannot be found, Claude must explicitly say so.

---

# Repository Discovery

Claude should inspect the repository for:

* programming languages
* application entry points
* configuration
* dependency manifests
* Docker configuration
* infrastructure configuration
* database access
* cache usage
* message brokers
* event producers
* event consumers
* background workers
* scheduled jobs
* external APIs
* tests
* retry mechanisms
* timeout handling
* transactions

Do not assume that a technology exists.

Only report technologies supported by repository evidence.

---

# Architecture Understanding

Claude should construct a lightweight mental model of the system.

For example:

```text
HTTP Request
    ↓
Order Service
    ↓
PostgreSQL
    ↓
Outbox
    ↓
Consumer
    ↓
Elasticsearch
```

This does not need to become a formal architecture diagram.

The goal is to understand where failures can propagate.

---

# Failure Boundaries

Pay particular attention to boundaries where:

* one service depends on another
* a network request can fail
* an event can be duplicated
* an event can be delayed
* an event can be reordered
* an operation can partially complete
* a process can crash between two side effects
* retries can repeat side effects
* stale data can be observed
* asynchronous processing can create eventual consistency
* external services can timeout
* databases can become unavailable
* caches can become unavailable
* consumers can fail
* multiple concurrent operations can race

---

# Failure Hypothesis Generation

A failure hypothesis should contain:

```text
Condition
+
Failure
+
Affected component
+
Potential consequence
```

Example:

```text
Condition:
An OrderCreated event is delivered twice.

Failure:
The consumer processes both deliveries.

Potential consequence:
The order side effect may happen twice.
```

The hypothesis must be connected to actual repository evidence.

---

# Scenario Selection

Generate several possible hypotheses internally.

Then select the **5 strongest scenarios**.

Prefer scenarios that are:

1. Strongly supported by repository evidence.
2. Technically plausible.
3. Relevant to important system behavior.
4. Specific to this repository.
5. Potentially reproducible in a future chaos experiment.

Do not generate five variations of the same failure.

Try to cover different failure classes when the repository supports them.

---

# Evidence Requirement

Every scenario must contain concrete repository evidence.

Good:

```text
Evidence:

internal/events/order_consumer.go

The consumer processes OrderCreated events without
an apparent idempotency check.
```

Bad:

```text
Payment systems often have duplicate-event problems.
```

Generic statements are not evidence.

If there is insufficient evidence, say:

```text
Evidence insufficient.
```

Never fabricate evidence.

---

# Fact vs Hypothesis

The report must distinguish between facts and hypotheses.

### Fact

```text
The consumer does not contain an apparent idempotency check.
```

### Hypothesis

```text
Duplicate delivery may cause duplicate processing.
```

### Confirmed Result

```text
The system processed the event twice.
```

Phase 0.1 performs no runtime chaos experiments.

Therefore Phase 0.1 cannot produce confirmed results.

Never state that the system definitely fails.

Use language such as:

* may
* could
* potential
* plausible
* likely, when evidence supports it

---

# Scenario Format

Each scenario should contain:

```text
Failure Type
Evidence
Failure Condition
Potential Consequence
How It Could Be Tested Later
Confidence
```

Example:

```markdown
## 1. Duplicate Event Processing

**Failure Type:** Messaging / Idempotency

**Evidence:**

`internal/events/order_consumer.go`

The consumer processes OrderCreated events and no apparent
idempotency check was found before the side effect.

**Failure Condition:**

The same event is delivered twice.

**Potential Consequence:**

The side effect may be executed twice.

**How It Could Be Tested Later:**

Deliver the same event twice and observe whether the side effect
occurs once or multiple times.

**Confidence:** High
```

---

# Confidence

Use only:

* High
* Medium
* Low

Confidence measures confidence in the **analysis**, not severity.

For example:

```text
High confidence:
The repository clearly shows the relevant code path.

Medium confidence:
The architecture suggests the failure, but some behavior is unclear.

Low confidence:
The scenario is plausible but repository evidence is incomplete.
```

Do not confuse confidence with severity.

---

# Report Format

The final output should be:

```markdown
# FaultScout Report

## Repository

<repository name>

## Architecture

<short architecture summary>

## Failure Scenarios

### 1. <scenario>

...

### 2. <scenario>

...

### 3. <scenario>

...

### 4. <scenario>

...

### 5. <scenario>

...

## Summary

<short summary of the most important reliability risks>
```

Keep the report concise.

A developer should be able to understand it in a few minutes.

---

# Example

Given a repository with:

```text
API
 ↓
PostgreSQL
 ↓
Outbox
 ↓
Kafka
 ↓
Consumer
 ↓
Elasticsearch
```

FaultScout might produce:

```text
# FaultScout Report

## Architecture

The system consists of an API backed by PostgreSQL.
Changes are propagated through an outbox/Kafka flow to an
Elasticsearch consumer.

## Failure Scenarios

### 1. Duplicate Event Processing

Evidence:
`internal/consumer/order.go`

No apparent idempotency mechanism was found.

Potential consequence:
The same event may produce duplicate side effects.

How to test later:
Deliver the same event twice.

Confidence:
High

### 2. Consumer Crash After Database Commit

Evidence:
`internal/consumer/order.go`

Database state is modified before the event acknowledgement.

Potential consequence:
A process crash may cause the message to be processed again.

How to test later:
Terminate the consumer after the database operation and before ACK.

Confidence:
Medium
```

The exact scenarios must depend on the actual repository.

---

# Important: Do Not Over-Engineer

This is an MVP.

Do not introduce:

* complex abstractions
* classes for every concept
* unnecessary configuration
* external dependencies
* frameworks
* databases
* APIs
* separate application layers

FaultScout V0.1 should primarily consist of **Claude Code instructions and skills**.

The intelligence should come from Claude's ability to inspect and reason about the repository.

---

# MCP — Future Phase

Do NOT implement MCP in Phase 0.1.

MCP becomes useful when FaultScout needs to perform actions that go beyond repository analysis.

Future MCP tools may include:

```text
get_service_status()
stop_service()
restart_service()
inject_latency()
disconnect_network()
collect_logs()
run_experiment()
```

The future architecture will become:

```text
Claude Code
     │
     ├── FaultScout Skills
     │
     └── FaultScout MCP
             │
             ├── Docker
             ├── Network
             ├── Services
             └── Experiments
```

MCP should provide capabilities.

Skills should define how Claude reasons and uses those capabilities.

Do not add MCP merely for the sake of using MCP.

---

# Future Roadmap

## Phase 0.1

Repository analysis.

```text
/chaos
   ↓
Analyze repository
   ↓
Find failure boundaries
   ↓
Generate 5 hypotheses
   ↓
Report
```

## Phase 0.2

Generate executable test scenarios.

## Phase 0.3

Introduce FaultScout MCP.

## Phase 0.4

Controlled local Docker failure injection.

## Phase 0.5

Execute chaos experiments.

```text
Hypothesis
    ↓
Failure Injection
    ↓
Test
    ↓
Observation
    ↓
Result
```

## Phase 1

CI integration.

```text
Pull Request
     ↓
FaultScout
     ↓
Generate scenarios
     ↓
Run experiments
     ↓
Detect regressions
     ↓
PR report
```

---

# Development Instructions for Claude

Before implementing anything:

1. Inspect the requested architecture.
2. Confirm that Phase 0.1 does not require Python.
3. Confirm that Phase 0.1 does not require MCP.
4. Keep the repository structure minimal.
5. Explain any deviation from the requested structure.
6. Implement incrementally.
7. Test the plugin structure.
8. Verify `/chaos` works as intended.

Do not silently expand the scope.

If you think a feature is necessary but it belongs to a later phase, explain it and leave it out.

The goal is not to build a large codebase.

The goal is to prove that:

> **Claude Code can use FaultScout to look at a real repository and discover meaningful, evidence-backed ways that system could fail.**
