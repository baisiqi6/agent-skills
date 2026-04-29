# Behavioral Patterns

## Table of Contents

1. [Chain of Responsibility](#chain-of-responsibility)
2. [Command](#command)
3. [Interpreter](#interpreter)
4. [Iterator](#iterator)
5. [Mediator](#mediator)
6. [Memento](#memento)
7. [Observer](#observer)
8. [State](#state)
9. [Strategy](#strategy)
10. [Template Method](#template-method)
11. [Visitor](#visitor)

---

## Chain of Responsibility

### Intent

Avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request. Chain the receiving objects and pass the request along the chain until an object handles it.

### When to Use

- Multiple objects may handle a request, and the handler isn't known a priori
- Want to issue a request to one of several objects without specifying the receiver explicitly
- The set of handlers should be specified dynamically
- Examples: logging levels, approval workflows, event bubbling, middleware pipelines

### When Not to Use

- The handler is always known in advance
- Only one handler exists
- Order doesn't matter and there's no "chain" semantics
- You need every handler to process the request (use Observer instead)

### Structure

- **Handler** — declares interface for handling requests; maintains successor link
- **ConcreteHandler** — handles requests it's responsible for; forwards to successor
- **Client** — initiates the request

### Pseudocode

```
abstract class Handler:
    protected next: Handler = null

    setNext(handler): 
        next = handler
        return handler  // enable fluent chaining

    handle(request):
        if next != null:
            return next.handle(request)
        return null

class AuthHandler extends Handler:
    handle(request):
        if not request.hasToken():
            return "Authentication failed"
        return super.handle(request)  // pass to next

class RateLimitHandler extends Handler:
    handle(request):
        if isRateLimited(request.ip):
            return "Rate limit exceeded"
        return super.handle(request)

class BusinessHandler extends Handler:
    handle(request):
        return processBusinessLogic(request)

// Setup chain
auth = new AuthHandler()
rateLimit = new RateLimitHandler()
business = new BusinessHandler()

auth.setNext(rateLimit).setNext(business)

// Usage
result = auth.handle(request)  // walks the chain
```

### Tradeoffs

[+] Decouples sender from receiver
[+] Handlers can be reordered, added, or removed dynamically
[+] Each handler focuses on one concern
[-] No guarantee the request will be handled
[-] Debugging can be hard — which handler processed it?
[-] Long chains may affect performance

### Related Patterns

- **Composite** — Chain of Responsibility is often applied to Composite structures
- **Command** — Command objects can be passed along a chain
- **Mediator** — Mediator centralizes communication; Chain of Responsibility decentralizes it
- **Observer** — Observer broadcasts to all; Chain passes to one

---

## Command

### Intent

Encapsulate a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.

### When to Use

- Parameterize objects with operations (callbacks, task queues)
- Support undo/redo
- Queue or schedule operations
- Log operations for recovery/audit
- Structure a system around high-level operations (transaction-like)

### When Not to Use

- Simple method calls suffice — no need for undo, queuing, or logging
- Only one operation exists
- The overhead of encapsulating each request as an object isn't justified

### Structure

- **Command** — declares interface for executing an operation
- **ConcreteCommand** — binds a receiver to an action
- **Receiver** — knows how to perform the operation
- **Invoker** — asks the command to carry out the request
- **Client** — creates ConcreteCommand and sets its receiver

### Pseudocode

```
interface Command:
    execute()
    undo()

// Receiver
class Editor:
    text: string = ""

    write(text):
        this.text += text

    deleteLast(n):
        removed = this.text[-n:]
        this.text = this.text[:-n]
        return removed

// Concrete Commands
class WriteCommand implements Command:
    private editor: Editor
    private text: string

    constructor(editor, text):
        this.editor = editor
        this.text = text

    execute():
        editor.write(text)

    undo():
        editor.deleteLast(text.length)

// Invoker
class EditorInvoker:
    private history: Stack<Command> = []

    executeCommand(cmd):
        cmd.execute()
        history.push(cmd)

    undo():
        if not history.isEmpty():
            history.pop().undo()

// Usage
editor = new Editor()
invoker = new EditorInvoker()

invoker.executeCommand(new WriteCommand(editor, "Hello "))
invoker.executeCommand(new WriteCommand(editor, "World"))
// editor.text == "Hello World"

invoker.undo()
// editor.text == "Hello "
```

### Tradeoffs

[+] Decouples invoker from receiver
[+] Supports undo/redo, queuing, logging
[+] Composable — macro commands from simple ones
[-] More classes for each operation
[-] Overhead for simple operations

### Related Patterns

- **Composite** — macro commands are composites of commands
- **Memento** — can store state for undo; Command stores the inverse operation
- **Strategy** — Strategy selects an algorithm; Command encapsulates an action
- **Prototype** — can clone commands for repetition

---

## Interpreter

### Intent

Given a language, define a representation for its grammar along with an interpreter that uses the representation to interpret sentences in the language.

### When to Use

- Need to evaluate expressions in a simple language or grammar
- Domain problem maps well to a grammar (arithmetic, search queries, rules)
- Grammar is simple and not performance-critical
- Rapid prototyping of a DSL

### When Not to Use

- Grammar is complex — use parser generators instead
- Performance matters — interpreted trees are slow
- A general-purpose language or existing DSL already solves the problem
- The grammar will change frequently

### Structure

- **AbstractExpression** — declares `interpret()` interface
- **TerminalExpression** — implements `interpret()` for terminal symbols
- **NonterminalExpression** — implements `interpret()` for non-terminals; contains sub-expressions
- **Context** — global state available during interpretation
- **Client** — builds the abstract syntax tree

### Pseudocode

```
interface Expression:
    interpret(context): bool

// Terminal expressions
class Variable implements Expression:
    private name: string

    constructor(name): this.name = name

    interpret(context): return context.get(name)

class Constant implements Expression:
    private value: bool

    constructor(value): this.value = value

    interpret(context): return value

// Non-terminal expressions
class And implements Expression:
    private left: Expression
    private right: Expression

    constructor(left, right):
        this.left = left; this.right = right

    interpret(context): return left.interpret(context) and right.interpret(context)

class Or implements Expression:
    private left: Expression
    private right: Expression

    constructor(left, right):
        this.left = left; this.right = right

    interpret(context): return left.interpret(context) or right.interpret(context)

class Not implements Expression:
    private expr: Expression

    constructor(expr): this.expr = expr

    interpret(context): return not expr.interpret(context)

// Usage — build AST: (A AND B) OR (NOT C)
context = {"A": true, "B": false, "C": true}
expr = new Or(
    new And(new Variable("A"), new Variable("B")),
    new Not(new Variable("C"))
)
result = expr.interpret(context)  // (true AND false) OR (NOT true) = false OR false = false
```

### Tradeoffs

[+] Easy to change/extend the grammar
[+] Grammar representation is intuitive (class per rule)
[+] Simple to implement for small grammars
[-] Doesn't scale to complex grammars (class explosion)
[-] Inefficient for large expression trees
[-] Rarely needed — parser generators are usually better

### Related Patterns

- **Composite** — abstract syntax tree is a Composite structure
- **Visitor** — can add operations to the AST without modifying expression classes
- **Flyweight** — share terminal expressions across the AST

---

## Iterator

### Intent

Provide a way to access the elements of an aggregate object sequentially without exposing its underlying representation.

### When to Use

- Need to traverse a collection without exposing its internal structure
- Multiple traversals of the same collection simultaneously
- Uniform traversal interface for different collection types
- Want to support different traversal strategies (in-order, pre-order, BFS, DFS)

### When Not to Use

- Only one collection type and one traversal order
- Language has built-in iteration (for-each, generators)
- Direct index access is sufficient and always will be

### Structure

- **Iterator** — interface for traversal (next, hasNext, current)
- **ConcreteIterator** — implements traversal for a specific aggregate
- **Aggregate** — interface for creating an Iterator
- **ConcreteAggregate** — returns a ConcreteIterator

### Pseudocode

```
interface Iterator:
    next(): Element
    hasNext(): bool
    current(): Element

interface IterableCollection:
    createIterator(): Iterator

class BinaryTree implements IterableCollection:
    root: TreeNode

    createIterator(): return new InOrderIterator(this)
    createPreOrderIterator(): return new PreOrderIterator(this)

class InOrderIterator implements Iterator:
    private stack: Stack<TreeNode>
    private current: TreeNode

    constructor(tree):
        pushLeftBranch(tree.root)

    private pushLeftBranch(node):
        while node != null:
            stack.push(node)
            node = node.left

    next():
        current = stack.pop()
        if current.right != null:
            pushLeftBranch(current.right)
        return current

    hasNext(): return not stack.isEmpty()

// Usage
tree = new BinaryTree(root)
iterator = tree.createIterator()
while iterator.hasNext():
    process(iterator.next())
```

### Tradeoffs

[+] Uniform traversal interface across collection types
[+] Multiple simultaneous traversals
[+] Separation of traversal logic from collection
[-] Overhead for simple collections
[-] Concurrent modification can invalidate iterator
[-] Most languages have built-in iteration

### Related Patterns

- **Composite** — iterators are often used to traverse Composite structures
- **Factory Method** — createIterator() is a Factory Method
- **Memento** — can snapshot iteration state

---

## Mediator

### Intent

Define an object that encapsulates how a set of objects interact. Promotes loose coupling by keeping objects from referring to each other explicitly.

### When to Use

- Set of objects communicate in complex but well-defined ways
- Reusing an object is difficult because it refers to many other objects
- Behavior distributed between classes should be customizable without subclassing
- UI components that interact (form validation, dialog coordination)

### When Not to Use

- Interactions are simple and few
- Objects don't need to know about each other
- The mediator would become a god object
- A simple event system or callback suffices

### Structure

- **Mediator** — interface for communicating with colleague objects
- **ConcreteMediator** — coordinates colleague objects; knows and maintains colleagues
- **Colleague** — communicates with other colleagues through mediator

### Pseudocode

```
interface ChatMediator:
    sendMessage(message, user)
    registerUser(user)

class ChatRoom implements ChatMediator:
    private users: List<User> = []

    registerUser(user):
        users.add(user)

    sendMessage(message, sender):
        for user in users:
            if user != sender:
                user.receive(message)

class User:
    private name: string
    private mediator: ChatMediator

    constructor(name, mediator):
        this.name = name
        this.mediator = mediator
        mediator.registerUser(this)

    send(message):
        mediator.sendMessage(message, this)

    receive(message):
        print(name + " received: " + message)

// Usage
mediator = new ChatRoom()
alice = new User("Alice", mediator)
bob = new User("Bob", mediator)
charlie = new User("Charlie", mediator)

alice.send("Hello everyone!")
// Bob and Charlie receive the message; Alice doesn't
```

### Tradeoffs

[+] Eliminates many-to-many connections between objects
[+] Centralizes communication logic
[+] Easier to understand object interactions
[-] Mediator can become overly complex (god object)
[-] Hard to debug — all communication goes through one point
[-] Mediator knows about all colleagues

### Related Patterns

- **Facade** — Facade is one-way (simplifies); Mediator is two-way (coordinates)
- **Observer** — Observer broadcasts events; Mediator handles specific interactions
- **Chain of Responsibility** — decentralized handling; Mediator is centralized
- **Colleague** objects can use **Observer** to notify the Mediator

---

## Memento

### Intent

Without violating encapsulation, capture and externalize an object's internal state so that the object can be restored to this state later.

### When to Use

- Need to save/restore object state (undo, snapshots, checkpoints)
- Direct access to object state would expose implementation details
- State history is needed (versioning, time-travel debugging)

### When Not to Use

- State is simple — just save the fields directly
- Performance is critical — deep copying is expensive
- State is very large — memory consumption becomes an issue
- You don't need to restore to previous states

### Structure

- **Memento** — stores internal state of the Originator
- **Originator** — creates a memento capturing its state; uses memento to restore
- **Caretaker** — keeps mementos safe; never examines contents

### Pseudocode

```
class Memento:
    private state: string  // opaque to caretaker

    constructor(state):
        this.state = state

    getState(): return state  // only Originator should call this

class Editor:  // Originator
    private text: string = ""
    private cursor: int = 0

    type(content):
        text += content
        cursor = text.length

    save(): 
        return new Memento(text + "|" + cursor)

    restore(memento):
        parts = memento.getState().split("|")
        text = parts[0]
        cursor = int(parts[1])

class History:  // Caretaker
    private stack: Stack<Memento> = []

    push(memento):
        stack.push(memento)

    pop():
        return stack.pop()

// Usage
editor = new Editor()
history = new History()

editor.type("Hello")
history.push(editor.save())

editor.type(" World")
history.push(editor.save())

editor.type("!!!")

// Undo
editor.restore(history.pop())  // "Hello World"
editor.restore(history.pop())  // "Hello"
```

### Tradeoffs

[+] Preserves encapsulation — state saved without exposing internals
[+] Simple undo/rollback mechanism
[+] Caretaker manages history without knowing state details
[-] Memory consumption for large or frequent snapshots
[-] Deep copying complex object graphs is expensive
[-] Managing memento lifecycle (when to discard) adds complexity

### Related Patterns

- **Command** — can use Memento for undo; Command stores inverse operation, Memento stores state
- **Iterator** — Memento can snapshot iteration state
- **Prototype** — deep clone can serve as a memento alternative

---

## Observer

### Intent

Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

### When to Use

- Change to one object requires changing others, and you don't know how many need changing
- An object should notify others without being tightly coupled to them
- Event handling, pub/sub, data binding, reactive systems
- Multiple views of the same data model (MVC)

### When Not to Use

- Only one observer exists — use a direct callback
- Observers must be notified synchronously in a specific order
- The notification chain creates circular dependencies
- Performance is critical and broadcast overhead is unacceptable

### Structure

- **Subject** — knows its observers; provides interface for attaching/detaching
- **Observer** — interface for objects that should be notified
- **ConcreteSubject** — stores state of interest; sends notifications
- **ConcreteObserver** — maintains reference to Subject; implements update

### Pseudocode

```
interface Observer:
    update(data)

interface Subject:
    attach(observer)
    detach(observer)
    notify()

class WeatherStation implements Subject:
    private observers: List<Observer> = []
    private temperature: float

    attach(observer): observers.add(observer)
    detach(observer): observers.remove(observer)

    notify():
        for observer in observers:
            observer.update(temperature)

    setTemperature(temp):
        temperature = temp
        notify()  // push model

class PhoneDisplay implements Observer:
    private station: WeatherStation

    constructor(station):
        this.station = station
        station.attach(this)

    update(temperature):
        render("Phone: " + temperature + "°C")

class WindowDisplay implements Observer:
    update(temperature):
        render("Window: " + temperature + "°C")

// Usage
station = new WeatherStation()
phone = new PhoneDisplay(station)
window = new WindowDisplay()
station.attach(window)

station.setTemperature(25.0)
// Both PhoneDisplay and WindowDisplay are updated
```

### Tradeoffs

[+] Loose coupling between subject and observers
[+] Dynamic subscription — add/remove observers at runtime
[+] Broadcast communication — subject doesn't know observer details
[-] Unexpected update chains (observer triggers another notification)
[-] Memory leaks — observers not detached from subjects
[-] Update order is unpredictable
[-] Debugging notification flows can be hard

### Related Patterns

- **Mediator** — Mediator centralizes; Observer decentralizes notification
- **Strategy** — Strategy changes behavior; Observer reacts to state
- **Singleton** — Subject is often a Singleton (global event bus)
- **Command** — Commands can be used to represent notifications

---

## State

### Intent

Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.

### When to Use

- Object's behavior depends on its state, and it must change at runtime
- Operations have large, multi-branch conditionals that depend on state
- State transitions are complex with many possible states
- Examples: order status, media player (playing/paused/stopped), TCP connection

### When Not to Use

- Only two states with simple transitions — an enum + if/else suffices
- State doesn't affect behavior
- State machine is trivial and won't grow

### Structure

- **State** — interface for state-specific behavior
- **ConcreteState** — implements state-specific behavior
- **Context** — maintains current state; delegates to state object

### Pseudocode

```
interface OrderState:
    next(context: Order)
    cancel(context: Order)
    status(): string

class NewOrder implements OrderState:
    next(context):
        context.setState(new PaymentPending())
    cancel(context):
        context.setState(new Cancelled())
    status(): return "New"

class PaymentPending implements OrderState:
    next(context):
        context.setState(new Shipped())
    cancel(context):
        context.setState(new Cancelled())
    status(): return "Payment Pending"

class Shipped implements OrderState:
    next(context):
        context.setState(new Delivered())
    cancel(context):
        throw "Cannot cancel shipped order"
    status(): return "Shipped"

class Delivered implements OrderState:
    next(context):
        throw "Order is complete"
    cancel(context):
        throw "Cannot cancel delivered order"
    status(): return "Delivered"

class Cancelled implements OrderState:
    next(context):
        throw "Order is cancelled"
    cancel(context):
        throw "Already cancelled"
    status(): return "Cancelled"

// Context
class Order:
    private state: OrderState = new NewOrder()

    setState(state):
        this.state = state

    proceed():
        state.next(this)

    cancel():
        state.cancel(this)

    status():
        return state.status()

// Usage
order = new Order()
order.status()       // "New"
order.proceed()      // → PaymentPending
order.proceed()      // → Shipped
order.cancel()       // throws "Cannot cancel shipped order"
```

### Tradeoffs

[+] Eliminates large conditional statements
[+] State-specific logic is localized — easy to add new states
[+] State transitions are explicit
[-] More classes — one per state
[-] State objects may need access to Context (coupling)
[-] Overkill for simple state machines

### Related Patterns

- **Strategy** — same structure; Strategy changes algorithm, State changes behavior based on state
- **Flyweight** — State objects are often Flyweights (shared across contexts)
- **Singleton** — State objects are often Singletons (no instance-specific data)
- **Interpreter** — can model state machines with a grammar

---

## Strategy

### Intent

Define a family of algorithms, encapsulate each one, and make them interchangeable. Lets the algorithm vary independently from clients that use it.

### When to Use

- Need to switch algorithms at runtime
- Many related classes differ only in behavior
- Algorithm variants should not be exposed to client
- A class has many behaviors encoded in conditional statements

### When Not to Use

- Only one algorithm exists and won't change
- The algorithm is trivial — a function parameter suffices
- Context doesn't need to switch strategies dynamically

### Structure

- **Strategy** — interface for the algorithm
- **ConcreteStrategy** — implements the algorithm
- **Context** — configured with a ConcreteStrategy; delegates to it

### Pseudocode

```
interface SortStrategy:
    sort(data): List

class BubbleSort implements SortStrategy:
    sort(data):
        // bubble sort implementation
        return sortedData

class QuickSort implements SortStrategy:
    sort(data):
        // quick sort implementation
        return sortedData

class MergeSort implements SortStrategy:
    sort(data):
        // merge sort implementation
        return sortedData

class Sorter:  // Context
    private strategy: SortStrategy

    constructor(strategy):
        this.strategy = strategy

    setStrategy(strategy):
        this.strategy = strategy

    sort(data):
        return strategy.sort(data)

// Usage
sorter = new Sorter(new BubbleSort())
sorter.sort(smallData)       // bubble sort for small datasets

sorter.setStrategy(new QuickSort())
sorter.sort(largeData)       // quick sort for large datasets
```

### Tradeoffs

[+] Algorithms vary independently from context
[+] Eliminates conditional statements for selecting behavior
[+] Easy to add new strategies (Open/Closed Principle)
[+] Strategies can be swapped at runtime
[-] Client must be aware of strategy differences to choose
[-] Some strategies may not use all context data
[-] Increased number of objects

### Related Patterns

- **State** — same structure; State changes behavior based on state, Strategy changes algorithm
- **Template Method** — Template Method uses inheritance; Strategy uses composition
- **Flyweight** — Strategy objects are often Flyweights (stateless, shared)
- **Decorator** — Decorator changes appearance; Strategy changes internals

---

## Template Method

### Intent

Define the skeleton of an algorithm in an operation, deferring some steps to subclasses. Lets subclasses redefine certain steps without changing the algorithm's structure.

### When to Use

- Fixed algorithm structure with variable steps
- Common behavior across subclasses should be factored into a common class
- Want to control extension points (hook methods)
- Frameworks where the framework calls your code (inversion of control)

### When Not to Use

- Algorithm has no common structure across variants
- Steps vary so much that the "template" is meaningless
- Composition (Strategy) would be more flexible

### Structure

- **AbstractClass** — defines abstract primitive operations; implements template method
- **ConcreteClass** — implements primitive operations

### Pseudocode

```
abstract class DataMiner:
    // Template method — defines the algorithm skeleton
    mine(path):
        data = openFile(path)
        rawData = extractData(data)
        parsedData = parseData(rawData)
        analysis = analyzeData(parsedData)
        sendReport(analysis)
        closeFile(data)

    // Abstract steps — subclasses must implement
    abstract extractData(data)
    abstract parseData(rawData)

    // Common implementations
    openFile(path):
        return FileSystem.open(path)

    analyzeData(data):
        return StatisticalAnalysis.run(data)

    sendReport(analysis):
        Email.send("report@example.com", analysis)

    closeFile(data):
        data.close()

    // Hook method — optional override
    customizeReport(report):
        return report  // default: no customization

class PDFDataMiner extends DataMiner:
    extractData(data):
        return PDFParser.extract(data)

    parseData(rawData):
        return PDFParser.parse(rawData)

class CSVDataMiner extends DataMiner:
    extractData(data):
        return CSVReader.read(data)

    parseData(rawData):
        return CSVParser.parse(rawData)

    customizeReport(report):
        return report + " (CSV source)"

// Usage
pdfMiner = new PDFDataMiner()
pdfMiner.mine("report.pdf")  // algorithm runs with PDF-specific steps

csvMiner = new CSVDataMiner()
csvMiner.mine("data.csv")    // same algorithm, CSV-specific steps
```

### Tradeoffs

[+] Reuses common code in the base class
[+] Controls extension points — subclasses can only override intended steps
[+] Clear algorithm structure
[-] Inheritance-based — less flexible than Strategy
[-] Violates Liskov Substitution if hook methods have undocumented expectations
[-] Hard to maintain as the number of steps grows

### Related Patterns

- **Strategy** — Strategy uses composition; Template Method uses inheritance
- **Factory Method** — a specialization of Template Method for object creation
- **Command** — Command encapsulates an operation; Template Method structures an algorithm

---

## Visitor

### Intent

Represent an operation to be performed on the elements of an object structure. Lets you define a new operation without changing the classes of the elements.

### When to Use

- Need to add operations to a stable object structure without modifying it
- Object structure is composed of many classes with different interfaces
- Operations depend on concrete types, not just abstract interfaces
- Related operations should be grouped together (e.g., export, validation, optimization)

### When Not to Use

- Object structure changes frequently — every new element type requires changing all visitors
- Only one or two operations are needed
- Operations are naturally part of the element classes
- Double dispatch isn't needed (single dispatch suffices)

### Structure

- **Visitor** — declares a visit() method for each ConcreteElement type
- **ConcreteVisitor** — implements each visit() with operation-specific logic
- **Element** — declares accept(Visitor) method
- **ConcreteElement** — implements accept() by calling visitor.visit(this)
- **ObjectStructure** — can enumerate elements; may be a Composite

### Pseudocode

```
interface ShapeVisitor:
    visitCircle(circle)
    visitRectangle(rectangle)
    visitTriangle(triangle)

interface Shape:
    accept(visitor)

class Circle implements Shape:
    radius: float
    x: float
    y: float

    accept(visitor):
        visitor.visitCircle(this)  // double dispatch

class Rectangle implements Shape:
    width: float
    height: float

    accept(visitor):
        visitor.visitRectangle(this)

class Triangle implements Shape:
    base: float
    height: float

    accept(visitor):
        visitor.visitTriangle(this)

// Concrete Visitor — area calculation
class AreaCalculator implements ShapeVisitor:
    totalArea: float = 0

    visitCircle(circle):
        totalArea += PI * circle.radius * circle.radius

    visitRectangle(rect):
        totalArea += rect.width * rect.height

    visitTriangle(tri):
        totalArea += 0.5 * tri.base * tri.height

// Concrete Visitor — XML export
class XMLExporter implements ShapeVisitor:
    xml: string = ""

    visitCircle(circle):
        xml += "<circle r='" + circle.radius + "'/>"

    visitRectangle(rect):
        xml += "<rect w='" + rect.width + "' h='" + rect.height + "'/>"

    visitTriangle(tri):
        xml += "<triangle base='" + tri.base + "' height='" + tri.height + "'/>"

// Usage
shapes = [new Circle(5), new Rectangle(3, 4), new Triangle(6, 2)]

areaCalc = new AreaCalculator()
for shape in shapes:
    shape.accept(areaCalc)
print(areaCalc.totalArea)

exporter = new XMLExporter()
for shape in shapes:
    shape.accept(exporter)
print(exporter.xml)
```

### Tradeoffs

[+] Add operations without modifying element classes (Open/Closed Principle)
[+] Related logic is grouped in one visitor class
[+] Can accumulate state during traversal
[-] Adding new element types requires changing all visitors
[-] Breaks encapsulation — visitors need access to element internals
[-] Element structure must be stable; frequent changes make Visitor impractical
[-] Double dispatch is unfamiliar to many developers

### Related Patterns

- **Composite** — Visitor is often used to operate on Composite structures
- **Interpreter** — Visitor can add operations to the AST without modifying expression classes
- **Iterator** — can traverse the object structure; Visitor applies operations
- **Command** — Visitor focuses on structure traversal; Command encapsulates single operations
