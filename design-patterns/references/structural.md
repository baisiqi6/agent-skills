# Structural Patterns

## Table of Contents

1. [Adapter](#adapter)
2. [Bridge](#bridge)
3. [Composite](#composite)
4. [Decorator](#decorator)
5. [Facade](#facade)
6. [Flyweight](#flyweight)
7. [Proxy](#proxy)

---

## Adapter

### Intent

Convert the interface of a class into another interface clients expect. Lets classes work together that couldn't otherwise because of incompatible interfaces.

### When to Use

- Need to use an existing class whose interface doesn't match your needs
- Want to create a reusable class that cooperates with unrelated or unforeseen classes
- Integrating with a legacy system or third-party library
- Need to adapt several sources with different interfaces to a uniform target

### When Not to Use

- You control both sides and can just change one interface
- A simple wrapper function or method suffices
- The mismatch is trivial (e.g., one method renaming)

### Structure

- **Target** — the interface the client expects
- **Adapter** — translates Target interface to Adaptee interface
- **Adaptee** — the existing interface that needs adapting

### Pseudocode

```
// Target interface (what the client expects)
interface Logger:
    logInfo(message)
    logError(message)

// Adaptee (existing third-party logger with different API)
class ThirdPartyLogger:
    writeLog(level, message)

// Adapter
class LoggerAdapter implements Logger:
    private adaptee: ThirdPartyLogger

    constructor(adaptee):
        this.adaptee = adaptee

    logInfo(message):
        adaptee.writeLog("INFO", message)

    logError(message):
        adaptee.writeLog("ERROR", message)

// Client uses Target interface, unaware of Adaptee
class Client:
    private logger: Logger

    doWork():
        logger.logInfo("Starting work")
        logger.logError("Something went wrong")
```

### Tradeoffs

[+] Makes incompatible interfaces work together without modifying either
[+] Single responsibility — adapter handles only interface translation
[+] Can adapt multiple adaptees to one target
[-] Adds an indirection layer
[-] Can obscure the real behavior of the adaptee

### Related Patterns

- **Bridge** — similar structure but different intent (Bridge separates up front; Adapter retrofits)
- **Decorator** — adds behavior without changing interface; Adapter changes interface
- **Facade** — simplifies interface; Adapter makes existing interface match a target
- **Proxy** — same interface; Adapter provides a different interface

---

## Bridge

### Intent

Decouple an abstraction from its implementation so that the two can vary independently.

### When to Use

- Want to avoid a permanent binding between abstraction and implementation
- Abstraction and implementation should be extensible by subclassing
- Changes in implementation should not affect client code
- Need to share an implementation among multiple objects
- Want to switch implementations at runtime

### When Not to Use

- Only one implementation exists and won't change
- No need for platform independence or implementation swapping
- The abstraction and implementation are tightly coupled by design

### Structure

- **Abstraction** — defines abstraction interface, holds reference to Implementor
- **RefinedAbstraction** — extends Abstraction
- **Implementor** — defines implementation interface
- **ConcreteImplementor** — implements Implementor

### Pseudocode

```
// Implementor
interface Renderer:
    renderCircle(radius)
    renderSquare(side)

// Concrete Implementors
class VectorRenderer implements Renderer:
    renderCircle(radius):  // draw vector circle
    renderSquare(side):    // draw vector square

class RasterRenderer implements Renderer:
    renderCircle(radius):  // draw raster circle
    renderSquare(side):    // draw raster square

// Abstraction
class Shape:
    protected renderer: Renderer

    constructor(renderer):
        this.renderer = renderer

    abstract draw()

// Refined Abstractions
class Circle extends Shape:
    private radius: float

    constructor(renderer, radius):
        super(renderer)
        this.radius = radius

    draw():
        renderer.renderCircle(radius)

class Square extends Shape:
    private side: float

    constructor(renderer, side):
        super(renderer)
        this.side = side

    draw():
        renderer.renderSquare(side)

// Usage — same shape, different renderers
vectorCircle = new Circle(new VectorRenderer(), 5)
rasterCircle = new Circle(new RasterRenderer(), 5)
```

### Tradeoffs

[+] Abstraction and implementation vary independently
[+] Eliminates permanent binding
[+] Implementation can change at runtime
[+] Clean separation of concerns
[-] Adds complexity for simple cases
[-] May be overkill if only one implementation exists

### Related Patterns

- **Adapter** — retrofits compatibility; Bridge designs for it up front
- **Strategy** — Bridge is similar to Strategy at the implementation level; Bridge emphasizes ongoing structural separation
- **Abstract Factory** — can create and configure Bridge implementations

---

## Composite

### Intent

Compose objects into tree structures to represent part-whole hierarchies. Lets clients treat individual objects and compositions uniformly.

### When to Use

- Represent a part-whole hierarchy (files/folders, UI containers, org charts)
- Want clients to ignore the difference between individual and composite objects
- Structure is recursive — composites can contain other composites

### When Not to Use

- Structure is flat with no nesting
- Individual and composite objects need fundamentally different APIs
- You need to enforce type safety that "leaf" vs "composite" distinction provides

### Structure

- **Component** — declares interface for objects in the composition
- **Leaf** — represents leaf objects (no children)
- **Composite** — represents composite objects (can have children)

### Pseudocode

```
abstract class Graphic:
    abstract draw()

class Line extends Graphic:  // Leaf
    draw():  // draw a line

class Rectangle extends Graphic:  // Leaf
    draw():  // draw a rectangle

class Picture extends Graphic:  // Composite
    private children: List<Graphic> = []

    add(graphic):
        children.add(graphic)

    remove(graphic):
        children.remove(graphic)

    draw():
        for child in children:
            child.draw()  // uniform treatment

// Usage
root = new Picture()
root.add(new Line())
root.add(new Rectangle())

subPicture = new Picture()
subPicture.add(new Line())
subPicture.add(new Rectangle())

root.add(subPicture)  // composite in composite
root.draw()           // draws everything recursively
```

### Tradeoffs

[+] Uniform treatment of individual and composite objects
[+] Easy to add new component types
[+] Simplifies client code — no type checks needed
[-] Can over-generalize (hard to restrict components)
[-] Hard to enforce "only leaves" or "only composites" constraints
[-] Debugging deep composites can be difficult

### Related Patterns

- **Visitor** — add operations to composite structures
- **Decorator** — similar recursive structure; Decorator adds behavior, Composite adds children
- **Iterator** — traverse composite structures
- **Builder** — often used to build Composite structures

---

## Decorator

### Intent

Attach additional responsibilities to an object dynamically. Provides a flexible alternative to subclassing for extending functionality.

### When to Use

- Add responsibilities to individual objects, not to an entire class
- Extend functionality without modifying existing code (Open/Closed Principle)
- Stack behaviors (e.g., compression → encryption → logging)
- Subclassing is impractical (too many combinations)

### When Not to Use

- Only one decoration is needed and won't change
- Deep decoration chains become confusing
- Need to inspect or remove specific decorations at runtime
- The base component is a simple data object

### Structure

- **Component** — interface for objects that can have responsibilities added
- **ConcreteComponent** — the original object
- **Decorator** — wraps a Component and delegates to it
- **ConcreteDecorator** — adds responsibilities

### Pseudocode

```
interface DataSource:
    writeData(data)
    readData(): string

class FileDataSource implements DataSource:  // ConcreteComponent
    private filename: string

    constructor(filename):
        this.filename = filename

    writeData(data):  // write to file
    readData():  // read from file

class DataSourceDecorator implements DataSource:  // Base Decorator
    protected wrappee: DataSource

    constructor(source):
        this.wrappee = source

    writeData(data): wrappee.writeData(data)
    readData(): return wrappee.readData()

class EncryptionDecorator extends DataSourceDecorator:
    writeData(data):
        wrappee.writeData(encrypt(data))

    readData():
        return decrypt(wrappee.readData())

class CompressionDecorator extends DataSourceDecorator:
    writeData(data):
        wrappee.writeData(compress(data))

    readData():
        return decompress(wrappee.readData())

// Usage — stack decorators
source = new CompressionDecorator(
    new EncryptionDecorator(
        new FileDataSource("data.txt")
    )
)
source.writeData("sensitive data")
// Writes: compress(encrypt("sensitive data"))
```

### Tradeoffs

[+] More flexible than inheritance — add/remove at runtime
[+] Follows Open/Closed Principle
[+] Multiple decorators can be combined in any order
[-] Many small objects — harder to understand the stack
[-] Decorator identity differs from component identity (type checks fail)
[-] Ordering of decorators can matter but isn't enforced

### Related Patterns

- **Adapter** — changes interface; Decorator keeps interface and adds behavior
- **Composite** — similar recursive wrapping; Composite manages children, Decorator delegates
- **Strategy** — Strategy changes internals; Decorator changes appearance/skin
- **Proxy** — similar wrapping structure; Proxy controls access, Decorator adds behavior

---

## Facade

### Intent

Provide a unified interface to a set of interfaces in a subsystem. Defines a higher-level interface that makes the subsystem easier to use.

### When to Use

- Need a simple interface to a complex subsystem
- Want to layer a system — facade defines entry point to each layer
- Decouple client code from subsystem internals
- Wrap a legacy or third-party API with a cleaner interface

### When Not to Use

- Subsystem is already simple enough
- Client needs fine-grained access that the facade hides
- Facade would become a "god object" that knows too much

### Structure

- **Facade** — knows which subsystem classes to delegate to
- **Subsystem classes** — implement functionality; unaware of Facade

### Pseudocode

```
// Complex subsystem
class Projector:
    on()
    off()
    setInput(source)
    wideScreenMode()

class Amplifier:
    on()
    off()
    setVolume(level)
    setSurroundSound()

class DVDPlayer:
    on()
    off()
    play(movie)
    stop()

class Lights:
    dim(level)
    on()

// Facade
class HomeTheaterFacade:
    private projector: Projector
    private amp: Amplifier
    private dvd: DVDPlayer
    private lights: Lights

    constructor(projector, amp, dvd, lights):
        ...

    watchMovie(movie):
        lights.dim(10)
        projector.on()
        projector.setInput("DVD")
        projector.wideScreenMode()
        amp.on()
        amp.setSurroundSound()
        amp.setVolume(5)
        dvd.on()
        dvd.play(movie)

    endMovie():
        dvd.stop()
        dvd.off()
        amp.off()
        projector.off()
        lights.on()

// Client — simple!
facade = new HomeTheaterFacade(projector, amp, dvd, lights)
facade.watchMovie("Inception")
facade.endMovie()
```

### Tradeoffs

[+] Simplifies client interaction with complex subsystems
[+] Decouples client from subsystem internals
[+] Doesn't prevent clients from using subsystem directly if needed
[-] Can become a "god object" if it accumulates too much logic
[-] May hide useful subsystem features behind a too-simple interface

### Related Patterns

- **Adapter** — adapts one interface; Facade simplifies many interfaces
- **Mediator** — Facade is one-way (client → subsystem); Mediator is two-way
- **Abstract Factory** — can be used with Facade to configure subsystems
- **Proxy** — Proxy controls access to one object; Facade simplifies access to many

---

## Flyweight

### Intent

Use sharing to support large numbers of fine-grained objects efficiently. Separate intrinsic (shared) state from extrinsic (unique) state.

### When to Use

- Application uses a large number of similar objects
- Storage cost is high due to quantity
- Most state can be made extrinsic (moved outside the shared object)
- Identity of each object doesn't matter — objects can be shared

### When Not to Use

- Objects are unique with little shared state
- Memory isn't a concern
- Can't cleanly separate intrinsic from extrinsic state
- Number of objects is small

### Structure

- **Flyweight** — interface through which flyweights can receive and act on extrinsic state
- **ConcreteFlyweight** — stores intrinsic state; implements Flyweight
- **FlyweightFactory** — creates and manages flyweight objects; ensures shared flyweights are reused
- **Client** — maintains extrinsic state

### Pseudocode

```
// Flyweight — stores intrinsic (shared) state
class TreeType:  // ConcreteFlyweight
    name: string
    color: string
    texture: string

    constructor(name, color, texture):
        ...

    draw(x, y):  // x, y are extrinsic state
        // render tree at position (x, y) using shared color/texture

// Flyweight Factory
class TreeFactory:
    private static treeTypes: Map<string, TreeType> = {}

    static getTreeType(name, color, texture):
        key = name + color + texture
        if not treeTypes.contains(key):
            treeTypes[key] = new TreeType(name, color, texture)
        return treeTypes[key]

// Client — extrinsic state stored here
class Tree:
    private x: int
    private y: int
    private type: TreeType  // shared flyweight

    constructor(x, y, type):
        ...

    draw():
        type.draw(x, y)

// Usage — 1,000,000 trees, but only a few TreeType objects
forest = []
oakType = TreeFactory.getTreeType("Oak", "green", "rough")
pineType = TreeFactory.getTreeType("Pine", "dark green", "smooth")

for i in range(500000):
    forest.add(new Tree(randomX(), randomY(), oakType))
for i in range(500000):
    forest.add(new Tree(randomX(), randomY(), pineType))
```

### Tradeoffs

[+] Massive memory savings when many objects share state
[+] Centralized management of shared objects
[-] Runtime cost to look up/manage shared instances
[-] Complexity of separating intrinsic from extrinsic state
[-] Thread-safety concerns for shared flyweights

### Related Patterns

- **Composite** — can share leaf nodes via Flyweight
- **State** — State objects are often Flyweights (shared across contexts)
- **Strategy** — Strategy objects can be Flyweights when they hold no state
- **Factory Method** — FlyweightFactory uses factory-like creation

---

## Proxy

### Intent

Provide a surrogate or placeholder for another object to control access to it.

### When to Use

- **Virtual Proxy** — lazy initialization of expensive objects
- **Protection Proxy** — control access based on permissions
- **Remote Proxy** — represent an object in a different address space
- **Smart Reference** — add actions on access (reference counting, logging)
- **Caching Proxy** — cache results of expensive operations

### When Not to Use

- Direct access works fine — no need for indirection
- The proxy adds no meaningful control or optimization
- A simple wrapper function suffices

### Structure

- **Subject** — interface for RealSubject and Proxy
- **RealSubject** — the real object the proxy represents
- **Proxy** — holds reference to RealSubject; controls access

### Pseudocode

```
interface Image:
    display()

class RealImage implements Image:  // expensive to construct
    private filename: string

    constructor(filename):
        loadFromDisk(filename)  // expensive operation

    display():
        // render image

class ImageProxy implements Image:
    private realImage: RealImage = null
    private filename: string

    constructor(filename):
        this.filename = filename

    display():
        if realImage == null:       // lazy initialization
            realImage = new RealImage(filename)
        realImage.display()

// Protection Proxy variant
class ProtectedImageProxy implements Image:
    private realImage: RealImage
    private user: User

    constructor(realImage, user):
        this.realImage = realImage
        this.user = user

    display():
        if user.hasPermission("view"):
            realImage.display()
        else:
            throw "Access denied"

// Usage — image loaded only when first displayed
image = new ImageProxy("large_photo.jpg")
// ... image not loaded yet
image.display()  // loaded and displayed on first access
```

### Tradeoffs

[+] Controlled access to the real object
[+] Lazy initialization reduces startup cost
[+] Additional functionality without changing the real object
[+] Client is unaware of the proxy
[-] Adds indirection — may introduce latency
[-] Response from proxy may be delayed (lazy loading)
[-] Code complexity increases

### Related Patterns

- **Adapter** — Adapter provides a different interface; Proxy provides the same interface
- **Decorator** — Decorator adds behavior; Proxy controls access
- **Facade** — Facade simplifies many interfaces; Proxy controls one
- **Flyweight** — Flyweight shares state; Proxy controls access
