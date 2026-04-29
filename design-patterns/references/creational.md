# Creational Patterns

## Table of Contents

1. [Singleton](#singleton)
2. [Factory Method](#factory-method)
3. [Abstract Factory](#abstract-factory)
4. [Builder](#builder)
5. [Prototype](#prototype)

---

## Singleton

### Intent

Ensure a class has only one instance and provide a global point of access to it.

### When to Use

- Exactly one instance must exist (e.g., configuration manager, connection pool, logger)
- Need a shared global state across the application
- Resource is expensive to duplicate (database connection, hardware interface)

### When Not to Use

- When you just want global variables — use a module/namespace instead
- In multithreaded contexts without careful synchronization
- When testing requires isolated instances (Singleton makes mocking hard)
- When dependency injection can achieve the same effect more cleanly

### Structure

- **Singleton** — class that holds a static instance and a static `getInstance()` method

### Pseudocode

```
class Singleton:
    private static instance: Singleton = null

    private constructor():  // prevent external instantiation
        pass

    static getInstance():
        if instance == null:
            instance = new Singleton()
        return instance

    someBusinessLogic():
        ...
```

Thread-safe variant:

```
class Singleton:
    private static instance: Singleton = null
    private static lock: Mutex

    private constructor():
        pass

    static getInstance():
        if instance == null:           // first check (no lock)
            acquire(lock)
            if instance == null:       // second check (with lock)
                instance = new Singleton()
            release(lock)
        return instance
```

### Tradeoffs

[+] Controlled access to sole instance
[+] Avoids global variables while providing similar convenience
[+] Lazy initialization possible
[-] Can hide bad design (excessive coupling to global state)
[-] Hard to test (shared state between tests)
[-] Double-checked locking is subtle and error-prone

### Related Patterns

- **Abstract Factory** — often implemented as Singleton
- **Builder** — may use Singleton for director
- **Prototype** — alternative to Singleton for managing shared prototypes

---

## Factory Method

### Intent

Define an interface for creating an object, but let subclasses decide which class to instantiate.

### When to Use

- A class cannot anticipate the class of objects it must create
- You want subclasses to choose which objects to create
- A framework needs to delegate object creation to framework users

### When Not to Use

- When the concrete type is known at compile time — just use `new`
- When there's only one variant and no foreseeable extension
- When a simple parameter-based switch suffices

### Structure

- **Product** — interface for the objects the factory creates
- **ConcreteProduct** — implements Product
- **Creator** — declares the factory method
- **ConcreteCreator** — overrides the factory method to return ConcreteProduct

### Pseudocode

```
interface Document:
    open()
    save()

class PdfDocument implements Document:
    open():  // PDF-specific
    save():  // PDF-specific

class WordDocument implements Document:
    open():  // Word-specific
    save():  // Word-specific

abstract class Application:
    abstract createDocument(): Document

    newDocument():
        doc = createDocument()
        doc.open()

class PdfApplication extends Application:
    createDocument(): return new PdfDocument()

class WordApplication extends Application:
    createDocument(): return new WordDocument()
```

### Tradeoffs

[+] Decouples client from concrete product classes
[+] Follows Open/Closed Principle — add products without changing existing code
[+] Consistent creation interface across variants
[-] May create many small creator subclasses
[-] Can be overkill for simple object creation

### Related Patterns

- **Abstract Factory** — creates families; often uses Factory Methods internally
- **Template Method** — Factory Method is a specialization of Template Method
- **Prototype** — no subclassing needed; clone instead of create

---

## Abstract Factory

### Intent

Provide an interface for creating families of related or dependent objects without specifying their concrete classes.

### When to Use

- System must be independent of how its products are created
- Products are designed to work together as a family
- You need to switch between product families at runtime or configuration time
- A library must provide objects without exposing their concrete types

### When Not to Use

- Products don't form a meaningful family
- Only one product variant exists
- The factory interface would have too many methods (consider splitting)

### Structure

- **AbstractFactory** — declares creation methods for each abstract product
- **ConcreteFactory** — implements creation methods for a specific family
- **AbstractProduct** — interface for a type of product
- **ConcreteProduct** — product belonging to a specific family
- **Client** — uses only AbstractFactory and AbstractProduct interfaces

### Pseudocode

```
interface Button:
    render()

interface Checkbox:
    render()

class MacButton implements Button:
    render():  // macOS style button

class MacCheckbox implements Checkbox:
    render():  // macOS style checkbox

class WindowsButton implements Button:
    render():  // Windows style button

class WindowsCheckbox implements Checkbox:
    render():  // Windows style checkbox

interface GUIFactory:
    createButton(): Button
    createCheckbox(): Checkbox

class MacFactory implements GUIFactory:
    createButton(): return new MacButton()
    createCheckbox(): return new MacCheckbox()

class WindowsFactory implements GUIFactory:
    createButton(): return new WindowsButton()
    createCheckbox(): return new WindowsCheckbox()

// Client
class Application:
    private factory: GUIFactory
    private button: Button

    constructor(factory):
        this.factory = factory

    createUI():
        button = factory.createButton()

// Usage
if platform == "mac":
    app = new Application(new MacFactory())
else:
    app = new Application(new WindowsFactory())
```

### Tradeoffs

[+] Ensures products from the same family are compatible
[+] Easy to switch between product families
[+] Clean separation of client from concrete products
[-] Hard to extend with new product types (must modify every factory)
[-] Can become bloated with many product families × product types

### Related Patterns

- **Factory Method** — often used inside Abstract Factory
- **Singleton** — ConcreteFactory is often a Singleton
- **Prototype** — can replace subclassing with cloning

---

## Builder

### Intent

Separate the construction of a complex object from its representation so that the same construction process can create different representations.

### When to Use

- Object has many optional parameters or configuration steps
- Need to create different representations of the same object
- Construction is complex (multi-step, conditional logic)
- Want immutable objects with many fields
- Telescoping constructor problem (too many constructor overloads)

### When Not to Use

- Object is simple with few fields — use a constructor or named parameters
- Only one representation exists and construction is trivial
- The "builder" would just be a wrapper around a constructor

### Structure

- **Builder** — interface for building parts of the product
- **ConcreteBuilder** — implements Builder; assembles the product
- **Director** — orchestrates building steps (optional)
- **Product** — the complex object being built

### Pseudocode

```
class House:
    walls: int
    doors: int
    windows: int
    hasGarage: bool
    hasPool: bool
    hasGarden: bool

class HouseBuilder:
    private house: House

    constructor():
        reset()

    reset():
        house = new House()

    setWalls(count): house.walls = count; return this
    setDoors(count): house.doors = count; return this
    setWindows(count): house.windows = count; return this
    addGarage(): house.hasGarage = true; return this
    addPool(): house.hasPool = true; return this
    addGarden(): house.hasGarden = true; return this

    build(): return house

// Fluent usage
house = new HouseBuilder()
    .setWalls(4)
    .setDoors(2)
    .setWindows(4)
    .addGarage()
    .addGarden()
    .build()

// Director (optional — for standard configurations)
class HouseDirector:
    buildSimpleHouse(builder):
        return builder.setWalls(4).setDoors(1).setWindows(2).build()

    buildLuxuryHouse(builder):
        return builder.setWalls(8).setDoors(4).setWindows(10)
            .addGarage().addPool().addGarden().build()
```

### Tradeoffs

[+] Constructs objects step by step — clear and readable
[+] Same construction process can produce different representations
[+] Eliminates telescoping constructors
[+] Fine control over construction process
[-] More code than a simple constructor
[-] Requires creating a builder class for each product
[-] Director adds another layer of indirection

### Related Patterns

- **Abstract Factory** — both create objects; Builder focuses on step-by-step construction, Abstract Factory on product families
- **Composite** — Builder often builds Composite structures
- **Singleton** — Director is sometimes a Singleton

---

## Prototype

### Intent

Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.

### When to Use

- System should be independent of how its products are created/represented
- Classes to instantiate are specified at runtime
- Objects are expensive to create (database queries, network calls, heavy computation)
- Need many similar objects that differ only slightly

### When Not to Use

- Objects are simple and cheap to construct
- Deep copying is problematic (circular references, shared resources)
- Each instance is truly unique with no shared structure

### Structure

- **Prototype** — declares a `clone()` method
- **ConcretePrototype** — implements `clone()` (deep or shallow copy)
- **Client** — creates new objects by asking a prototype to clone itself

### Pseudocode

```
abstract class Shape:
    x: int
    y: int
    color: string

    abstract clone(): Shape

    // Shallow copy base
    protected copyTo(target: Shape):
        target.x = this.x
        target.y = this.y
        target.color = this.color

class Circle extends Shape:
    radius: int

    clone():
        c = new Circle()
        copyTo(c)
        c.radius = this.radius
        return c

class Rectangle extends Shape:
    width: int
    height: int

    clone():
        r = new Rectangle()
        copyTo(r)
        r.width = this.width
        r.height = this.height
        return r

// Registry of prototypes
class ShapeRegistry:
    private prototypes: Map<string, Shape>

    register(key, prototype):
        prototypes[key] = prototype

    create(key):
        return prototypes[key].clone()

// Usage
registry = new ShapeRegistry()
registry.register("red-circle", new Circle(radius=10, color="red"))
registry.register("blue-rect", new Rectangle(width=20, height=10, color="blue"))

shape1 = registry.create("red-circle")   // cloned instance
shape2 = registry.create("red-circle")   // another cloned instance
```

### Tradeoffs

[+] Clone objects without coupling to their concrete classes
[+] Avoids expensive creation when a similar object exists
[+] Easy to add/remove prototypes at runtime
[+] Simplifies object creation for complex objects
[-] Deep vs shallow copy complexity
[-] Circular references complicate cloning
[-] Each subclass must implement clone()

### Related Patterns

- **Abstract Factory** — Prototype can replace Abstract Factory when product families are dynamic
- **Composite** — Prototype is useful for cloning Composite structures
- **Decorator** — Prototype can clone deeply nested decorator chains
