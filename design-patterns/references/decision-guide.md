# Decision Guide: Choosing the Right Pattern

Use this guide when you're unsure which pattern applies to a problem. Answer the questions in order to narrow down the candidates.

## Table of Contents

- [Step 1: Identify the Core Concern](#step-1-identify-the-core-concern)
- [Step 2: Creational Decision Tree](#step-2-creational-decision-tree)
- [Step 3: Structural Decision Tree](#step-3-structural-decision-tree)
- [Step 4: Behavioral Decision Tree](#step-4-behavioral-decision-tree)
- [Step 5: Disambiguating Similar Patterns](#step-5-disambiguating-similar-patterns)
- [Step 6: Combining Patterns](#step-6-combining-patterns)
- [Anti-Pattern Signals → Recommended Patterns](#anti-pattern-signals--recommended-patterns)
- [Code Review Checklist](#code-review-checklist)
- [Quick-Reference: Problem → Pattern](#quick-reference-problem--pattern)

---

## Step 1: Identify the Core Concern

What is the primary problem you're trying to solve?

| Core Concern | Go To |
|---|---|
| "How do I create or configure objects?" | Step 2 (Creational) |
| "How do I compose or structure classes/objects?" | Step 3 (Structural) |
| "How do I manage behavior, algorithms, or communication?" | Step 4 (Behavioral) |
| "I'm not sure" | Read all three trees and look for keywords that match your problem |

---

## Step 2: Creational Decision Tree

**Q: Do you need to control the number of instances?**
- Yes, exactly one → **Singleton**
- No, multiple is fine → Next question

**Q: Do you need to create a family of related products?**
- Yes, products that must work together → **Abstract Factory**
- No, individual products → Next question

**Q: Who decides which concrete class to instantiate?**
- A subclass should decide → **Factory Method**
- The client should decide → Next question

**Q: Is the object complex to construct (many parts, optional fields)?**
- Yes, build it step by step → **Builder**
- No, I want to clone an existing object → **Prototype**

---

## Step 3: Structural Decision Tree

**Q: What is the primary structural problem?**

- "I have incompatible interfaces" → **Adapter**
- "I want to add behavior without modifying the class" → **Decorator**
- "I need a simpler interface to something complex" → **Facade**

If none of the above:

**Q: Are you dealing with a tree or part-whole hierarchy?**
- Yes, treat individual and group uniformly → **Composite**
- No → Next question

**Q: Do you need to decouple an abstraction from its implementation?**
- Yes, so they can vary independently → **Bridge**
- No → Next question

**Q: Do you have many similar objects consuming too much memory?**
- Yes, share common state → **Flyweight**
- No → Next question

**Q: Do you need to control access to another object?**
- Yes, for lazy loading, access control, or caching → **Proxy**
- No → Reconsider: is this really a structural problem?

---

## Step 4: Behavioral Decision Tree

**Q: What is the primary behavioral problem?**

### Group A: Request handling
- "I need to decouple sender from receiver"
  - Chain of handlers, each may handle → **Chain of Responsibility**
  - Encapsulate the request as an object → **Command**
- "I need to interpret a language/grammar" → **Interpreter**

### Group B: Notification and state
- "I need to notify dependents when something changes" → **Observer**
- "Behavior should change when internal state changes" → **State**
- "I need to capture and restore state" → **Memento**

### Group C: Algorithm variation
- "I need to swap entire algorithms at runtime" → **Strategy**
- "I want to define an algorithm skeleton, let subclasses fill in steps" → **Template Method**

### Group D: Operations on structures
- "Add operations to objects without modifying their classes" → **Visitor**
- "Traverse a collection without exposing its structure" → **Iterator**

### Group E: Communication
- "Centralize complex communication between objects" → **Mediator**

---

## Step 5: Disambiguating Similar Patterns

Many problems can be addressed by multiple patterns. Use these distinctions to choose:

| Confused Pair | Key Question | If Yes → | If No → |
|---|---|---|---|
| **Strategy vs. State** | Are you swapping algorithms or responding to state transitions? | Strategy (algorithm choice) | State (behavior mode) |
| **Decorator vs. Proxy** | Are you adding behavior or controlling access? | Decorator (add behavior) | Proxy (control access) |
| **Adapter vs. Facade** | Are you matching an existing interface or creating a new simpler one? | Adapter (match existing) | Facade (create new) |
| **Adapter vs. Bridge** | Are you retrofitting compatibility or designing for it up front? | Adapter (retrofit) | Bridge (design up front) |
| **Factory Method vs. Abstract Factory** | One product type or a family of products? | Factory Method (one) | Abstract Factory (family) |
| **Command vs. Strategy** | Is this a reversible operation or a swappable algorithm? | Command (reversible) | Strategy (swappable) |
| **Composite vs. Decorator** | Are you building a tree or wrapping with behavior? | Composite (tree) | Decorator (wrap) |
| **Mediator vs. Observer** | Is communication centralized or broadcast? | Mediator (centralized) | Observer (broadcast) |
| **Memento vs. Command** | Do you need full state snapshots or incremental undo? | Memento (full snapshot) | Command (incremental) |
| **Builder vs. Abstract Factory** | Step-by-step construction or family creation? | Builder (step-by-step) | Abstract Factory (family) |
| **Iterator vs. Visitor** | Just traversing or adding operations? | Iterator (traverse) | Visitor (add operations) |
| **Template Method vs. Strategy** | Inheritance-based or composition-based? | Template Method (inheritance) | Strategy (composition) |

---

## Step 6: Combining Patterns

Patterns often work best together. Common combinations:

| Combination | Use Case |
|---|---|
| **Composite + Visitor** | Operate on tree structures without modifying node classes |
| **Composite + Iterator** | Traverse tree structures uniformly |
| **Strategy + Factory Method** | Create strategies via factory; swap at runtime |
| **Command + Memento** | Commands use mementos for undo/redo |
| **Command + Composite** | Macro commands from simple commands |
| **Observer + Mediator** | Mediator coordinates; observers react to changes |
| **Decorator + Composite** | Decorate individual or composite nodes |
| **Abstract Factory + Singleton** | Factory as a single shared instance |
| **Builder + Composite** | Build complex composite structures step by step |
| **State + Flyweight** | Share state objects across multiple contexts |
| **Bridge + Abstract Factory** | Factory creates implementation objects for Bridge |
| **Chain of Responsibility + Command** | Commands passed along a handler chain |

---

## Anti-Pattern Signals → Recommended Patterns

When reviewing code, look for these signals. Each suggests a specific pattern may help.

| Code Smell / Anti-Pattern Signal | Recommended Pattern | Example |
|---|---|---|
| Large `switch` or `if/else` on type | **Strategy** or **State** | Payment processing with switch on type → Strategy |
| `switch` on type to create objects | **Factory Method** | `if type == "A" return new A()` → Factory Method |
| Scattered `new` with type selection | **Factory Method** or **Abstract Factory** | Creating different DB connections in multiple places |
| Too many constructor parameters | **Builder** | `new Config(host, port, timeout, retry, debug, log, ...)` |
| Telescoping constructors (overloads) | **Builder** | Multiple constructors with different parameter combos |
| Clone-and-modify code duplication | **Prototype** | Copy-paste object creation with slight variations |
| Incompatible interface you can't change | **Adapter** | Third-party API with different method names |
| Explosion of subclasses for variations | **Bridge** or **Strategy** | Shape × Renderer = N×M subclasses → Bridge |
| Need to add behavior to individual objects | **Decorator** | Adding logging, compression, encryption to streams |
| Client struggling with complex subsystem | **Facade** | 5 classes to send an email → one EmailFacade |
| Many identical objects wasting memory | **Flyweight** | 10,000 tree objects with same texture → Flyweight |
| Expensive object not always needed | **Proxy** (virtual) | Large image loaded but maybe never displayed |
| Long chain of `if` for request handling | **Chain of Responsibility** | Auth → RateLimit → Validation → Handler |
| Need undo/redo | **Command** + **Memento** | Text editor undo stack |
| Tightly coupled notification logic | **Observer** | Direct calls to multiple objects on change |
| State-dependent behavior in conditionals | **State** | `if state == "playing" ... elif state == "paused" ...` |
| Algorithm variants in one class | **Strategy** | Sort method with type parameter → Strategy |
| Copy-pasted algorithm with minor steps changed | **Template Method** | Data mining pipeline with format-specific steps |
| Need operations on a stable object structure | **Visitor** | Export/validation/optimization on AST nodes |
| Many objects talking directly to each other | **Mediator** | UI components calling each other → ChatRoom mediator |
| Need to traverse collection without exposing it | **Iterator** | Custom tree traversal without exposing TreeNode |
| Global mutable state / god object | **Singleton** (if truly one) or **Mediator** or refactor | Consider if you really need it |

## Code Review Checklist

When reviewing code for pattern opportunities, follow this process:

### 1. Scan for Structural Signals
- [ ] Are there large conditional blocks (`switch`, `if/else` chains)?
- [ ] Is object creation scattered or duplicated?
- [ ] Are there many subclasses that differ only slightly?
- [ ] Is there a god class that does too much?
- [ ] Are objects directly referencing many other objects?

### 2. Identify the Pain Point
- **Hard to extend?** → Strategy, Template Method, Visitor
- **Hard to reuse?** → Factory Method, Abstract Factory, Bridge
- **Hard to understand?** → Facade, Mediator
- **Hard to change?** → Adapter, Decorator, Observer
- **Performance issues?** → Flyweight, Proxy, Prototype

### 3. Evaluate Whether a Pattern Helps
- Does the pattern solve the actual pain point?
- Is the added indirection justified by the benefit?
- Can the problem be solved more simply without a pattern?
- Will the codebase need this flexibility, or is it YAGNI?

### 4. Recommend with Context
- State the code smell you observed
- Name the pattern and explain why it fits
- Show the before/after structure (not full code)
- Note tradeoffs — every pattern has a cost

## Quick-Reference: Problem → Pattern

| Problem Phrase | Pattern(s) |
|---|---|
| "Only one instance allowed" | Singleton |
| "Create objects without knowing the class" | Factory Method, Abstract Factory |
| "Family of products that must work together" | Abstract Factory |
| "Complex object, many optional parts" | Builder |
| "Clone instead of creating" | Prototype |
| "Incompatible interfaces must cooperate" | Adapter |
| "Decouple abstraction from implementation" | Bridge |
| "Treat individual and group the same" | Composite |
| "Add behavior dynamically" | Decorator |
| "Simplify a complex subsystem" | Facade |
| "Too many similar objects, memory issue" | Flyweight |
| "Control access to another object" | Proxy |
| "Pass request along a chain" | Chain of Responsibility |
| "Encapsulate request as object" | Command |
| "Interpret a grammar" | Interpreter |
| "Traverse without exposing internals" | Iterator |
| "Centralize object communication" | Mediator |
| "Save and restore state" | Memento |
| "Notify when state changes" | Observer |
| "Change behavior on state change" | State |
| "Swap algorithms at runtime" | Strategy |
| "Algorithm skeleton with hook points" | Template Method |
| "Add operations without modifying classes" | Visitor |