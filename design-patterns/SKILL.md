---
name: design-patterns
description: GoF design pattern guidance for choosing, applying, reviewing, and refactoring creational, structural, and behavioral patterns. Use when the user asks about design patterns, pattern selection, object creation, code structure, decoupling, extensibility, architecture decisions, refactoring, code smells, large switch/case blocks, tight coupling, undo/redo, state machines, observers, strategies, factories, builders, adapters, decorators, facades, proxies, or how to improve code structure.
---

# Design Patterns — GoF 23 Patterns (Language-Agnostic)

## How to Use This Skill

This skill uses **progressive disclosure**:
- **This file**: Scene-based recommendation + quick reference (always loaded)
- **Reference files**: Deep-dive per category (loaded only when needed)
  - `references/creational.md` — 5 creational patterns
  - `references/structural.md` — 7 structural patterns
  - `references/behavioral.md` — 11 behavioral patterns
  - `references/decision-guide.md` — Decision tree & disambiguation

## Scene-Based Recommendation

| Problem Scenario | Recommended Pattern(s) |
|---|---|
| Need exactly one instance of a class | **Singleton** |
| Create objects without specifying exact class | **Factory Method** |
| Create families of related objects | **Abstract Factory** |
| Construct complex objects step by step | **Builder** |
| Clone objects instead of creating from scratch | **Prototype** |
| Make incompatible interfaces work together | **Adapter** |
| Decouple abstraction from implementation | **Bridge** |
| Treat individual and composite objects uniformly | **Composite** |
| Add responsibilities dynamically | **Decorator** |
| Simplify a complex subsystem interface | **Facade** |
| Share state among many similar objects | **Flyweight** |
| Control access to another object | **Proxy** |
| Chain handlers for a request | **Chain of Responsibility** |
| Encapsulate a request as an object | **Command** |
| Define a grammar and interpret sentences | **Interpreter** |
| Traverse a collection without exposing its structure | **Iterator** |
| Centralize complex interactions between objects | **Mediator** |
| Capture and restore object state | **Memento** |
| Notify dependents when state changes | **Observer** |
| Change behavior when internal state changes | **State** |
| Swap algorithms at runtime | **Strategy** |
| Define algorithm skeleton, let subclasses override steps | **Template Method** |
| Add operations to objects without modifying them | **Visitor** |

## Quick Reference

### Creational — Object creation mechanisms
- **Singleton** — Ensure a class has only one instance
- **Factory Method** — Delegate object creation to subclasses
- **Abstract Factory** — Create families of related objects
- **Builder** — Construct complex objects step by step
- **Prototype** — Clone existing objects

### Structural — Composition and relationships
- **Adapter** — Bridge incompatible interfaces
- **Bridge** — Decouple abstraction from implementation
- **Composite** — Uniform tree structure of objects
- **Decorator** — Attach responsibilities dynamically
- **Facade** — Simplified interface to a subsystem
- **Flyweight** — Share fine-grained state efficiently
- **Proxy** — Stand-in for another object

### Behavioral — Object interaction and responsibility
- **Chain of Responsibility** — Pass request along a handler chain
- **Command** — Encapsulate request as an object
- **Interpreter** — Define and evaluate a language grammar
- **Iterator** — Sequential access to collection elements
- **Mediator** — Centralize inter-object communication
- **Memento** — Snapshot and restore object state
- **Observer** — Publish-subscribe dependency notification
- **State** — Behavior changes with internal state
- **Strategy** — Encapsulate interchangeable algorithms
- **Template Method** — Algorithm skeleton with hook points
- **Visitor** — Separate operations from object structure

## Guiding Principles

1. **Don't force patterns** — A simple direct solution beats a pattern applied unnecessarily
2. **Start from the problem** — Identify the specific pain point, then find the pattern
3. **Understand tradeoffs** — Every pattern adds indirection; ensure the benefit outweighs the cost
4. **Combine when needed** — Patterns often work together (e.g., Factory Method + Template Method, Composite + Visitor)
5. **Prefer composition over inheritance** — Most structural and behavioral patterns embody this principle

## Deep-Dive Reference Files

When you need detailed structure, pseudocode, and tradeoffs for a specific pattern, load the appropriate reference file:

- **Creational patterns** → `references/creational.md`
- **Structural patterns** → `references/structural.md`
- **Behavioral patterns** → `references/behavioral.md`
- **Choosing between similar patterns** → `references/decision-guide.md`
