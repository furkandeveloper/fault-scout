# Security Policy

## Scope

FaultScout is a Claude Code plugin made of Markdown instructions only. It
contains no executable code, no dependencies, no network access of its own, and
never modifies the repository it analyzes. Its security surface is therefore the
**instructions themselves**: whether they could cause Claude to behave outside
the documented read-only boundaries.

We treat the following as security issues:

- an instruction that could lead Claude to modify, delete, or execute files in
  the analyzed repository
- an instruction that could lead Claude to run the project, its tests, or its
  build, or to start or stop processes, containers, or services
- an instruction that could lead Claude to contact external systems or exfiltrate
  repository contents
- a prompt-injection path by which content in an analyzed repository could
  redirect the analysis into any of the above

Inaccurate analysis results (a missed scenario, a wrong confidence grade) are
**bugs, not security issues** — please report those through a normal issue.

## Reporting a vulnerability

Please **do not** open a public issue for security concerns.

Use GitHub's private vulnerability reporting:
<https://github.com/gofabric/fault-scout/security/advisories/new>

Include the affected file and line, the behavior you observed or expect, and a
minimal way to reproduce it (for example, the content of a repository file that
triggers the problem). Anonymize anything sensitive.

You will receive an acknowledgement within 7 days. Fixes are released as a new
plugin version; the advisory is published once the fix is available.

## Supported versions

Only the latest released version receives fixes.
