## What

<!-- One or two sentences on the change. -->

## Why

<!-- The problem this solves or the report quality it improves. -->

## Checklist

- [ ] The change keeps FaultScout **read-only** and **analysis-only**
- [ ] `commands/analyze.md` and `skills/faultscout/SKILL.md` remain consistent
      (steps, section and field names, supported languages, boundaries)
- [ ] `claude plugin validate .` passes
- [ ] The forbidden-terms grep from `CONTRIBUTING.md` returns only intentional mentions
- [ ] If the version changed, both manifests were bumped together
- [ ] If analysis behavior changed, I ran it on a real repository and summarized
      the difference below

## Output difference (if applicable)

<!-- Anonymized before/after excerpt of the report. -->
