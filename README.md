# Agent Skills

A collection of skills for AI coding agents — language-agnostic design patterns and programming principles with progressive disclosure.

## Why This Exists

AI coding agents are fast and capable, but they share common failure modes:

- **Over-engineering** — Adding abstractions, patterns, and flexibility that nobody asked for
- **Under-engineering** — Writing happy-path-only code with no error handling or structural thinking
- **Pattern blindness** — Not recognizing when a design pattern would eliminate real complexity
- **Pattern shopping** — Applying patterns reflexively without identifying the actual pain point
- **Drive-by refactoring** — "Improving" adjacent code while making a targeted change

These skills give agents the judgment to navigate between "too simple" and "too abstract." They don't just list patterns — they teach **when to use them, when not to, and how to decide.**

## Skills

### design-patterns

GoF 23 classic design patterns with progressive disclosure. Includes scene-based recommendation, decision trees, anti-pattern signals, and pseudocode examples.

**Install:**
```bash
npx skills add baisiqi6/agent-skills@design-patterns -g -y
```

### programming-principles

A meta-skill that routes to and combines sub-skills. Resolves tension between "use a pattern" and "keep it simple."

**Install:**
```bash
npx skills add baisiqi6/agent-skills@programming-principles -g -y
```

**Dependencies:** This skill works best when [karpathy-guidelines](https://skills.sh/forrestchang/andrej-karpathy-skills) and `design-patterns` are also installed. It falls back to summarized guidance when they are unavailable.

## Install All

```bash
npx skills add baisiqi6/agent-skills@design-patterns -g -y
npx skills add baisiqi6/agent-skills@programming-principles -g -y
```

## Supported Agents

These skills work with any agent that reads from `~/.agents/skills/`, including:

- Claude Code
- Codex
- GitHub Copilot
- Cline
- Trae CN
- Amp
- Antigravity

## Example Prompts

| Prompt | Skill Triggered |
|---|---|
| "Which design pattern should I use for a notification system?" | design-patterns → Observer |
| "I have too many if/else branches for payment types" | design-patterns → Strategy |
| "How should I structure this code?" | programming-principles → Karpathy + Design Patterns |
| "Design an extensible plugin architecture" | programming-principles → Design Patterns first, simplicity as constraint |
| "Review this code for architecture issues" | programming-principles → Both sub-skills |
| "Keep it simple, don't over-engineer" | programming-principles → Karpathy Guidelines |

## Progressive Disclosure

Skills are designed with layered loading:

1. **SKILL.md** (always loaded) — Quick reference and scene-based recommendations
2. **Reference files** (loaded on demand) — Deep-dive with structure, pseudocode, and tradeoffs

This keeps the default context small while providing full detail when needed.

## License

MIT
