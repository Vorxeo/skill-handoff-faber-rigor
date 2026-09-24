# Handoff: Faber ↔ Rigor

Claude Code skill: `handoff-faber-rigor`

## What

Protocol that separates **building** (Faber) from **certifying** (Rigor). Faber delivers a handoff packet: intent, diff scope, Verified evidence, NOT tested, known risks. Rigor refuses fog and returns Approve / Approve-with-follow-ups / Request-changes with evidence. Faber does not self-certify.

## When to use

End of an implementation chunk before merge; authz / migration / contract / bench-affecting PRs; after adversarial-qa HOLE fixes before Approve.

## Install

Copy this folder into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/handoff-faber-rigor
cp SKILL.md ~/.claude/skills/handoff-faber-rigor/
```

Claude Code loads `SKILL.md` from `~/.claude/skills/<name>/`.

## Sibling skills

`verified-delivery`, `fail-closed-review`, `adversarial-qa`, `contract-and-compat`, `migration-and-data-safety`, `bench-loop`, `code-that-holds`, `guards-that-scan`, `karpathy-method`, `reachability-audit`

Proposed repos: see [Vorxeo](https://github.com/Vorxeo) `skill-*` packs.

## License

MIT — Copyright (c) 2026 Vorxeo. See [LICENSE](./LICENSE).
