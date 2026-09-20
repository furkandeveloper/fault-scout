# Contributing to FaultScout

Thanks for your interest in FaultScout. This document explains how to propose
changes and what keeps the project coherent.

## What FaultScout is (and is not)

FaultScout is a **Claude Code plugin** made entirely of instructions: one
command (`commands/analyze.md`) and one skill (`skills/faultscout/SKILL.md`).
It performs **read-only static reliability analysis** of a repository and prints
a report.

Contributions must keep it that way. A change is out of scope if it would:

- run, build, or test the analyzed project
- inject failures or manipulate processes, containers, networks, caches,
  databases, or brokers
- add MCP servers, hooks, agents, backend services, dashboards, or runtime code
  in any language
- modify the analyzed repository automatically
- present a scenario as tested, reproduced, or confirmed

If you are unsure whether an idea fits, open a discussion or a feature request
first — it saves everyone time.

## Good first contributions

- **Report quality feedback.** Run `/faultscout:analyze` on a real codebase and
  tell us where the report was wrong, vague, padded, or missed something
  obvious. Anonymize anything sensitive. This is the most valuable input we
  get.
- **Evidence and hedging rules.** Tighten wording in `SKILL.md` so that claims
  are better grounded or easier to self-check.
- **Failure classes.** Propose a class that is missing, with the concrete code
  patterns that would prove it.
- **Report languages.** Add a heading/label table to the skill and the value to
  the supported list in both the skill and the command.
- **Documentation.** Clarify the README or this file.

## Development setup

There is no build step and no dependency. You need Claude Code and a checkout.

```bash
git clone https://github.com/gofabric/fault-scout.git
cd fault-scout
claude plugin validate .
```

Test your change against a real repository by loading the checkout as a
plugin:

```bash
cd /path/to/some/project
claude --plugin-dir /path/to/fault-scout
# then, inside Claude Code:
/faultscout:analyze
```

Changes to the command or skill files take effect on the next session.

## Before opening a pull request

1. Keep the command and the skill consistent: same steps, same section and
   field names, same supported languages, same boundaries.
2. Keep `README.md`, `CLAUDE.md`, `.claude-plugin/plugin.json`, and
   `.claude-plugin/marketplace.json` describing the same product and the same
   version.
3. Run:

   ```bash
   claude plugin validate .
   grep -rniE "mcp|experiment|inject|snapshot|restore|chaos-analysis|chaos-experiment|faultscout:chaos" \
     --include='*.md' --include='*.json' .
   ```

   Validation must pass. The grep must return only intentional mentions (for
   example, the README stating that FaultScout does *not* inject failures).
4. If you changed analysis behavior, run the plugin on at least one real
   repository and summarize in the PR what changed in the output.

## Pull request guidelines

- One logical change per PR.
- Explain *why* in the description, not only *what*.
- For skill changes, quote the before/after wording of the rule you touched.
- Do not paste real reports from private codebases; anonymize or use a public
  project.

## Versioning

Versions follow `MAJOR.MINOR.PATCH`. Bump `version` in **both** manifests in the
same commit. Changes that alter the report structure or scenario fields are at
least a minor bump.

## Code of conduct

This project follows the [Contributor Covenant](CODE_OF_CONDUCT.md). By
participating you agree to uphold it.

## License

By contributing you agree that your contributions are licensed under the
[MIT License](LICENSE).
