---
name: programming-principles
description: Programming principles meta-skill for coding best practices, design patterns, code quality, refactoring, code review, and software architecture. Use this whenever the user asks how to structure code, which design pattern to use, whether code is over-engineered, how to refactor safely, or how to make an engineering tradeoff. This skill routes to and combines Karpathy coding guidelines for simplicity and surgical execution with GoF design patterns for structural decisions; use both when practical coding discipline and architecture choices overlap.
---

# Programming Principles

This is a meta-skill. It does not replace the narrower skills; it decides when to load them, how to combine them, and how to resolve tension between "use a pattern" and "keep it simple."

## Sub-Skills

This skill can bridge to other installed skills, but it must remain useful without machine-specific paths. When a referenced skill is available in the current environment, use it. When it is not available, fall back to the summarized guidance in this file.

### 1. Karpathy Coding Guidelines

**Load when:** The task involves implementation, refactoring, code review, debugging, adding tests, reducing complexity, or deciding whether an abstraction is justified.

**Preferred skill:** `karpathy-guidelines`

**Covers:**
- Think first, ask questions before assuming
- Simplest solution that works — no premature abstractions
- Precise, minimal changes — no drive-by refactoring
- Define verifiable success criteria

### 2. GoF Design Patterns

**Load when:** The task involves object creation, decoupling, extensibility, state transitions, strategy selection, request pipelines, undo/redo, observers/events, adapters/facades, decorators/proxies, or choosing/reviewing a named design pattern.

**Preferred skill:** `design-patterns`

**Covers:**
- 23 classic design patterns (creational, structural, behavioral)
- Scene-based pattern recommendation
- Decision tree for choosing between patterns
- Anti-pattern signals -> recommended patterns
- Code review checklist for pattern opportunities

## Routing Guide

| User Intent | Load Sub-Skill |
|---|---|
| "How should I write this code?" | Karpathy Guidelines |
| "How should I structure this code?" | Karpathy Guidelines first; Design Patterns if structure is the main issue |
| "Keep it simple, don't over-engineer" | Karpathy Guidelines |
| "Which design pattern should I use?" | Design Patterns |
| "Design an architecture for me" | Design Patterns first; Karpathy as constraint against over-design |
| "Design an extensible system" | Design Patterns first; Karpathy as constraint |
| "Design a plugin system / framework" | Design Patterns first; Karpathy as constraint |
| "Review this code for architecture/patterns" | Both |
| "This code smells / too many if-else" | Karpathy Guidelines first; Design Patterns if conditionals model real variation |
| "Refactor this safely" | Karpathy Guidelines; Design Patterns only if the current shape needs a structural pattern |
| General coding best practices | Karpathy Guidelines; add Design Patterns based on the concrete task |

## Combination Rules

When this skill is triggered:

1. Identify the primary concern from the routing guide.
2. If the relevant preferred sub-skill is available, load it before giving detailed advice or making edits.
3. Apply Karpathy Guidelines as the default baseline for implementation discipline.
4. Apply Design Patterns only after a concrete structural pain has been identified.
5. If a preferred sub-skill is not available, do not block. Use the summarized rules in this file and mention that the deeper sub-skill was not available only when that affects confidence.

When both Karpathy Guidelines and Design Patterns apply, use this order:

1. Start with the simplest direct design.
2. Identify the pain that the direct design fails to handle.
3. Use Design Patterns to name one or two candidate structures.
4. Reject candidates whose indirection is larger than the current pain.
5. Choose the smallest pattern-shaped implementation that fits the existing codebase.
6. Define verification before or alongside the change.

## Conflict Resolution

If the pattern guidance suggests a larger abstraction but the simplicity guidance suggests a direct fix, prefer the direct fix unless at least one of these is true:

- The code already has multiple concrete variants that need the same interface.
- The current implementation has duplicated branching that is already hard to change safely.
- The abstraction matches an existing local pattern in the codebase.
- The user explicitly asked for a reusable architecture, library surface, or framework-level design.

If none of those are true, mention the possible pattern as a future option, but implement or recommend the simpler design.

## Output Expectations

For design advice:

- Give the recommendation first.
- Say which sub-skill guidance was used.
- Explain the concrete pain and why the chosen structure fits it.
- Include the smallest viable implementation shape.
- Include a verification plan.

For implementation:

- Make surgical edits.
- Avoid speculative extension points.
- Run checks proportional to risk.
- Summarize what changed, what was verified, and what remains unverified.

For code review:

- Lead with bugs, regressions, missing tests, and maintainability risks.
- Mention design pattern opportunities only when they address a real issue.
- Avoid style-only findings unless they affect correctness or consistency.
