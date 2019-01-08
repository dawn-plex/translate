> * 原文地址：[Keep it Simple with the Strategy Design Pattern](https://blog.bitsrc.io/keep-it-simple-with-the-strategy-design-pattern-c36a14c985e9)
> * 原文作者：[Chidume Nnamdi](https://blog.bitsrc.io/@kurtwanger40)
> * 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
> * 译文链接：[https://github.com/dawn-plex/translate/blob/master/articles/Understanding-Currying-in-JavaScript.md](https://github.com/dawn-plex/translate/blob/master/articles/Understanding-Currying-in-JavaScript.md)
> * 译者：[灵沼](https://github.com/su-dan)
> * 校对者：[也树](https://github.com/xdlrt)，[照天](https://github.com/zzwzzhao)

---

# Keep it Simple with the Strategy Design Pattern
[Chidume Nnamdi](https://blog.bitsrc.io/@kurtwanger40)

bject-oriented programming is a programming paradigm that revolves around using objects and declaration of Classes to provide simple and reusable design to our program.

According to Wikipedia:
> “Object-oriented programming (OOP) is a programming paradigm based on the concept of “objects”, which may contain data, in the form of fields, often known as attributes; and code, in the form of procedures, often known as methods.”

Wow!! nice but OOP is not all, how to structure your classes and their relationships are what matters. Complex systems like the brain, city, anthill, buildings are full of patterns. To achieve a long-lasting state they are built with a well-structured architecture. And software development is not left out.

Designing a large application requires intricate and complex connection and collaboration of objects and data.

OOP provides the design to do that, but as I said earlier we need a pattern to achieve a long-lasting one. Problems might arise in our OOP-designed app which might lead to decay.

As such, these problems have been cataloged overtime and elegant solutions for each of them have been described by experienced early Software Developers. These solutions are known as the Design Patterns.

To date, there are 24 design patterns, as described in the original book, `Design Patterns: Elements of Reusable Object-Oriented Software`. Each of these patterns provides a set of solution to a particular problem.

In this article, we will look into the **Strategy Pattern** to understand how it works and how and when to apply it during software development.

Tip: Build Javascript apps faster with [Bit](https://github.com/teambit/bit). Easily share components across projects and apps, collaborate with your team and use them like Lego. It’s a great way to improve modularity and keep your code DRY at scale.

## Strategy Pattern: Basic Idea
Strategy Design Pattern is a type of behavioral design patterns that encapsulates a "family" of algorithms and selects one from the pool for use during runtime. The algorithms are interchangeable, meaning that they are substitutable for each other.

> The strategy pattern is a behavioral design pattern that enables selecting an algorithm at runtime — Wikipedia

The key idea is to create objects which represent various strategies. These objects form a pool of strategies from which the context object can choose from to vary its behavior as per its strategy. These objects(strategies) perform the same operation, have the same(single) job and compose the same interface strategy.

Let’s take the sorting algorithms we have for example. Sorting algorithms have a set of rule specific to each other they follow to effectively sort an array of numbers. We have the

* Bubble Sort
* Linear Search
* Heap Sort
* Merge Sort
* Selection Sort

to name a few.

Then, in our program, we need different sorting algorithms at a time during execution. Using SP allows us to group these algorithms and select from the pool when needed.

It is more like a plugin, like the PlugnPlay in Windows or in the Device Drivers. All the plugins must follow a signature or rule.

For example, a Device Driver could be anything, Battery Driver, Disk Driver, Keyboard Driver …

They must implement:

```
NTSTATUS DriverEntry (_In_ PDRIVER_OBJECT ob, _In_ PUNICODE_STRING pstr) {
    //...
}
VOID DriverUnload(PDRIVER_OBJECT DriverObject)
{
    RtlFreeUnicodeString(&servkey);
}
NTSTATUS AddDevice(PDRIVER_OBJECT DriverObject, PDEVICE_OBJECT pdo)
{
    return STATUS_SOMETHING; // e.g., STATUS_SUCCESS
}
```

Every driver must implement the above functions, the DriverEntry is used by the OS when loading a driver, the DriverUnload when removing the driver from memory, the AddDriver for adding the driver to the driver list.

The OS doesn’t need to know what your driver does, all it knows that since you called it a driver it will assume all those are present and will call them at the required time.

If we lump the sorting algorithms in one class we will find ourselves writing conditional statements to select one algorithm.

Most importantly, all the strategies must have the same signature. If you are using an OO-Language make sure the strategies inherit from a common interface, if using a non-OO-Language like JavaScript, make sure the strategies have a common method to call by the context.

```
// In an OOP Language -
// TypeScript
// interface all sorting algorithms must implement
interface SortingStrategy {
    sort(array);
}
// heap sort algorithm implementing the `SortingStrategy` interface, it implements its algorithm in the `sort` method
class HeapSort implements SortingStrategy {
    sort() {
        log("HeapSort algorithm")
        // implementation here
    }
}
// linear search sorting algorithm implementing the `SortingStrategy` interface, it implements its algorithm in the `sort` method
class LinearSearch implements SortingStrategy {
    sort(array) {
        log("LinearSearch algorithm")
        // implementation here
    }
}
class SortingProgram {
    private sortingStrategy: SortingStrategy
    constructor(array: Array<Number>) {
    }
    runSort(sortingStrategy: SortingStrategy) {
        return this.sortingStrategy.sort(this.array)
    }
}
// instantiate the `SortingProgram` with an array of numbers
const sortProgram = new SortingProgram([9,2,5,3,8,4,1,8,0,3])
// sort using heap sort
sortProgram.runSort(new HeapSort())
// sort using linear search
sortProgram.runSort(new LinearSearch())
```

The SortingProgram takes a SortingStrategy as param in its runSort and calls the sort method. Any concrete implementation of the `SortingStrategy` must implement the `sort` method.

You see SP supports the [SOLID principles](https://blog.bitsrc.io/solid-principles-every-developer-should-know-b3bfa96bb688) and forces us to abide by it. The D in SOLID says we must depend on abstractions, not on concretions. That’s what happened in the runSort method. Also the O, which says entities should be open for, not extension.

If we had taken an alternative of subclassing our sorting algorithms, we will eventually run into a code that is hard to understand and maintain because we will have many related classes with the difference being on the algorithms they carry. The I, we have one specific interface for the concrete strategy to implement.

It isn’t bogus just specific to the job because any sorting algorithm will have to run the sort to sort:). The S, all the classes implementing the strategy have only one job of sorting. The L, all subclasses of the concrete strategies are substitutable for their superclasses.

## Structure
![](https://cdn-images-1.medium.com/max/1600/1*4vdmSjQVWuBF7C2ZGelfYA.png)

In the figure above, the `Context` class depends on the `Strategy`. During execution or runtime, different strategies of `Strategy` type are passed to the `Context` class. The Strategy provides the template by which the strategies must abide by for implementation.

![](https://cdn-images-1.medium.com/max/2000/1*9hN9PDGic_nmL4VOL-B73Q.png)

In the above UML class diagram, the Concrete class depends on an abstraction, Strategy interface. It doesn’t implement the algorithm directly. The Context from its method `runStraegy` calls the doAlgorithm in the Strategy concretion passed to it. The Context class is independent of the method and doesn't know and doesn't need to know how the doAlgorithm method is implemented. By virtue of `Design by Contract`, the class implementing the Strategy interface must implement the doAlgorithm method.

In strategy design pattern, there are three main entities: Context, Strategy, and ConcreteStrategy.

The **Context** is the body composing the concrete strategies where they play out their roles.

**Strategy** is the template that defines how all startegies must be configured.

**ConcreteStrategy** is the implementation of the Strategy template(interface).

## Examples
Using Steve Fenton’s example `Car Wash program`, you know car wash can run on different grades of washing and cleaning depending on the money the driver has, the more the money the higher the wash level. Let's the Car Wash offers:

* Basic Wheel and Body washing
* Executive Wheel and Body washing

The Basic wheel and Body cleaning is just the normal soaping and rinsing for the body and brushing for the car.

Executive cleaning goes beyond that, they wax the body and the wheel to make it look shiny and then dry them. The cleaning depends on the level the driver pays for. level 1 gives you Basic cleaning for both body and wheels:

```
interface BodyCleaning {
    clean(): void;
}
interface WheelCleaning {
    clean(): void;
}
class BasicBodyCleaningFactory implements BodyCleaning {
    clean() {
        log("Soap Car")
        log("Rinse Car")
    }
}
class ExecutiveBodyCleaningFactory implements BodyCleaning {
    clean() {
        log("Wax Car")
        log("Blow-Dry Car")
    }
}
class BasicWheelCleaningFactory implements BodyCleaning {
    clean() {
        log("Soap Wheel")
        log("Rinse wheel")
    }
}
class ExecutiveWheelCleaningFactory implements BodyCleaning {
    clean() {
        log("Brush Wheel")
        log("Dry Wheel")
    }
}
class CarWash {
    washCar(washLevel: Number) {
        switch(washLevel) {
            case 1: 
                    new BasicBodyCleaningFactory().clean()
                    new BasicWheelCleaningFactory().clean()
                break;
            case 2: 
                    new BasicBodyCleaningFactory().clean()
                    new ExecutiveWheelCleaningFactory().clean()
                break;
            case 3: 
                    new ExecutiveBodyCleaningFactory().clean()
                    new ExecutiveWheelCleaningFactory().clean()
                break;
        }
    }
}
```

You see now, some pattern is emerging. We are reusing the same class in many conditions, the classes are related but differ in behavior. Also, our code is getting untidy and heavy.

Most importantly, this our program fails the Open-Closed Principle in the S.O.L.I.D principles, which states that modules should be open for `extension` not `modification`.

For every new wash level, another conditional is added, that’s modification.

Using the strategy pattern, we will have to relieve our CarWash program of any responsibility for our knowledge of water level.

To do that we have to separate the cleaning actions. First, we create an interface all actions must implement:

```
interface ValetFaactory {
    getWheelCleaning();
    getBodyCleaning();
}
```

Then all the cleaning strategies:

```
class BronzeWashFactory implements ValetFactory {
    getWheelCleaning() {
        return new BasicWheelCleaning();
    }
    getBodyCleaning() {
        return new BasicBodyCleaning();
    }
}
class SilverWashFactory implements ValetFactory {
    getWheelCleaning() {
        return new BasicWheelCleaning();
    }
    getBodyCleaning() {
        return new ExecutiveBodyCleaning();
    }
}
class GoldWashFactory implements ValetFactory {
    getWheelCleaning() {
        return new ExecutiveWheelCleaning();
    }
    getBodyCleaning() {
        return new ExecutiveBodyCleaning();
    }
}
```

Next, we touch the CarWash program:

```
// ...
class CarWashProgram {
    constructor(private cleaningFactory: ValetFactory) {
    }
    runWash() {
        const wheelWash = this.cleaningFactory.getWheelCleaning();
        wheelWash.cleanWheels();
        
        const bodyWash = this.cleaningFactory.getBodyCleaning();
        bodyWash.cleanBody();
    }
}
```

Now, we pass any cleaning strategy we want to the CarWashProgram.

```
// ...
const carWash = new CarWashProgram(new GoldWashFactory())
carWash.runWash()
const carWash = new CarWashProgram(new BronzeWashFactory())
carWash.runWash()
```
## Another Example: Authentication Strategy
Let’s say we have an app, that we want to secure ie add authentication to it. We have different auth schemes and strategies:

* Basic
* Digest
* OpenID
* OAuth

We might try to implement something like this:

```
class BasicAuth {}
class DigestAuth {}
class OpenIDAuth {}
class OAuth {}
class AuthProgram {
    runProgram(authStrategy:any, ...) {
        this.authenticate(authStrategy)
        // ...
    }
    authenticate(authStrategy:any) {
        switch(authStrategy) {
            if(authStrategy == "basic")
                useBasic()
            if(authStrategy == "digest")
                useDigest()
            if(authStrategy == "openid")
                useOpenID()
            if(authStrategy == "oauth")
                useOAuth()
        }
    }
}
```

The same old long chain of conditionals. Also, if we want to auth. for a particular route in our program, we will find ourselves with the same thing.

```
class AuthProgram {
    route(path:string, authStyle: any) {
        this.authenticate(authStyle)
        // ...
    }
}
```

If we apply the strategy design pattern here, we will create an interface that all auth strategies must implement:

```
interface AuthStrategy {
    auth(): void;
}
class Auth0 implements AuthStrategy {
    auth() {
        log('Authenticating using Auth0 Strategy')
    }
}
class Basic implements AuthStrategy {
    auth() {
        log('Authenticating using Basic Strategy')
    }
}
class OpenID implements AuthStrategy {
    auth() {
        log('Authenticating using OpenID Strategy')
    }
}
```

The AuthStrategy defines the template by which all strategies must build on. Any concrete auth strategy must implement the auth method to provide us with its style of authentication. We have the Auth0, Basic and OpenID concrete strategies.

Next, we need to touch our AuthProgram class:

```
// ...
class AuthProgram {
    private _strategy: AuthStrategy
    use(strategy: AuthStrategy) {
        this._strategy = strategy
        return this
    }
    authenticate() {
        if(this._strategy == null) {
            log("No Authentication Strategy set.")
        }
        this._strategy.auth()
    }
    route(path: string, strategy: AuthStrategy) {
        this._strategy = strategy
        this.authenticate()
        return this
    }
}
```

You see now, the authenticate method doesn’t carry the long switch case. The use method sets the authentication strategy to use and the authenticate method just calls the auth method. It cares less about how the AuthStrategy implements its authentication.

```
log(new AuthProgram().use(new OpenID()).authenticate())
// Authenticating using OpenID Strategy
```

## Strategy Pattern: Problems It Solves
Strategy Pattern prevents hard-wiring of all the algorithms into the program. This makes our program complex and much more bogus and hard to refactor/maintain and understand.

This, in turn, makes our program to contain algorithms they do not use.

Let’s say we have a Printer class that prints in different flavors and style. If we contain all the styles and flavors of printing into the Printer class:

```
class Document {...}
class Printer {
    print(doc: Document, printStyle: Number) {
        if(printStyle == 0 /* color printing*/) {
            // ...
        }
        if(printStyle == 1 /* black and white printing*/) {
            // ...            
        }
        if(printStyle == 2 /* sepia color printing*/) {
            // ...
        }
        if(printStyle == 3 /* hue color printing*/) {
            // ...            
        }
        if(printStyle == 4 /* oil printing*/) {
            // ...
        }
        // ...
    }
}
```

OR

```
class Document {...}
class Printer {
    print(doc: Document, printStyle: Number) {
        switch(printStyle) {
            case 0 /* color priniting strategy*/:
                ColorPrinting()
                break;
            case 0 /* color priniting strategy*/:
                InvertedColorPrinting()
                break;
            // ...
        }
        // ...
    }
}
```

You see we end up with a bogus class, that is hard to read, maintain and with too many conditionals.

But with the Strategy Pattern, we break the printing styles into different tasks.

```
class Document {...}
interface PrintingStrategy {
    printStrategy(d: Document): void;
}
class ColorPrintingStrategy implements PrintingStrategy {
    printStrategy(doc: Document) {
        log("Color Printing")
        // ...
    }
}
class InvertedColorPrintingStrategy implements PrintingStrategy {
    printStrategy(doc: Document) {
        log("Inverted Color Printing")
        // ...
    }
}
class Printer {
    private printingStrategy: PrintingStrategy
    print(doc: Document) {
        this.printingStrategy.printStrategy(doc)
    }
}
```

So, instead of many conditionals, each condition is moved to a separate strategy class. There is no need for the Printer class to know the different printing styles implementation.

## Strategy Pattern and the SOLID Principles
In Strategy Pattern, composition is used over inheritance. It is advised to program to abstraction than to concretions. You see that Strategy Pattern is compatible with [the SOLID principles](https://blog.bitsrc.io/solid-principles-every-developer-should-know-b3bfa96bb688).

As an example, we have a DoorProgram that have different styles of locking mechanism to lock doors. As different locking mechanisms change between subclasses of door. We might be tempted to apply the door locking mechanism to the `Door` class like this:

```
class Door {
    open() {
        log('Opening Door')
        // ...
    }
    lock() {
        log('Locking Door')
    }
    lockingMechanism() {
        // card swipe
        // thumbprint
        // padlock
        // bolt
        // retina scanner
        // password
    }
}
```

It seems OK, but the behaviours of doors differs. Each has its own locking and opening mechanism. That is different behaviours.

When we create different types of Doors:

```
// ...
class TimedDoor extends Door {
    open() {
        super.open()
    }
}
```

And try to implement it the open/lock-ing mechanism, you see that we must call the parent method before implementing its own open/lock mechanism.

If we make the `Door` an interface like this:

```
interface Door { 
    open()
    lock()
}
```

You see that the open/lock behavior must be declared in each class or model or types of Door.

```
class GlassDoor implements Door {
    open() {
        // ...
    }
    lock() {
        // ...
    }
}
```

Quite good, but there are many drawbacks here which will pop up as our app grows. A Door model must have an open/lock mech. Is it a must a Door must open/close? No. A Door might not even be closed at all. So we see our Door models will be `forced` to open/lock.

Next, the interface doesn't draw a line between using the interface as a model or as an open/lock mech. Note: in [S in SOLID](https://blog.bitsrc.io/solid-principles-every-developer-should-know-b3bfa96bb688) a class must have one responsibility.

A Glass Door must have the only characteristics of a Glass Door also a Wooden Door, a Metal Door, a Ceramic Door(do they have that?) Another class should be responsible for handling the opening/locking mechanism.

Using SP, we separate our related, in this case, the locking/opening mech. into classes. Then at runtime, we pass the Door model the lock/open mechanism it is to use. The Door model can select from a pool of lock/open strategies which lock/open mech. to use.

```
interface LockOpenStrategy {
    open();
    lock();
}
class RetinaScannerLockOpenStrategy implements LockOpenStrategy {
    open() {
        //...
    }
    lock() {
        //...
    }
}
class KeypadLockOpenStrategy implements LockOpenStrategy {
    open() {
        if(password != "nnamdi_chidume"){
            log("Entry Denied")
            return
        }
        //...
    }
    lock() {
        //...
    }
}
abstract class Door {
    public lockOpenStrategy: LockOpenStrategy
}
class GlassDoor extends Door {}
class MetalDoor extends Door {}
class DoorAdapter {
    openDoor(d: Door) {
        d.lockOpenStrategy.open()
    }
}
const glassDoor = new GlassDoor()
glassDoor.lockOpenStrategy = new RetinaScannerLockOpenStrategy();
const metalDoor = new MetalDoor()
metalDoor.lockOpenStrategy = new KeypadLockOpenStrategy();
new DoorAdapter().openDoor(glassDoor)
new DoorAdapter().openDoor(metalDoor)
```

Each open/lock strategy is defined in a class inheriting from a base interface. SP supports this because it is better to code to an interface so as to achieve high cohesion.

Next, we have our Door models each a subclass of the Door class. We have a DoorAdapter whose job is to open doors passed to it. We created objects of a couple of Door models and set their lock/open strategies. The glass door is to be locked/opened via retina scanning and the metal door has a keypad for entering the secret password.

The thing we achieved here is the separation of concerns, separation of related behaviors. Each Door models doesn’t know and bear the concern of implementing a certain locking/opening strategy, it was delegated to another entity. We programmed to an interface as required by SP because it makes switching strategies during runtime easy.

This might not hold for long but it is a better approach courtesy of the Strategy Pattern.

A Door might have many lock/open strategies and might use one or all during both locking and opening. Whatever you do keep the Strategy Pattern in mind.

## Strategy Pattern in JavaScript
Most of our examples are based on OOP languages. JS isn’t statically typed but dynamically typed. So there is no concept of the OOP like interface, polymorphism, encapsulation, delegation is not present. But in SP, we can assume they are present, we simulate them.

Let’s use our first example to demonstrate how we could apply SP in JS.

The first example was based on sorting algorithms. Now, the interface SortingStrategy has a method sort that all implementing strategies must define. The SortingProgram class takes a SortingStrategy in its runSort method and calls the sort method.

We model our sorting algorithms:

```
var HeapSort = function() {
    this.sort(array) {
        log("HeapSort algorithm")
        // implementation here
    }
}
// linear search sorting algorithm implementing its alogrithm in the `sort` method
var LinearSearch = function() {
    this.sort(array) {
        log("LinearSearch algorithm")
        // implementation here
    }
}

class SortingProgram {
    constructor(array) {
        this.array=array
    }
    runSort(sortingStrategy) {
        return sortingStrategy.sort(this.array)
    }
}
// instantiate the `SortingProgram` with an array of numbers
const sortProgram = new SortingProgram([9,2,5,3,8,4,1,8,0,3])
// sort using heap sort
sortProgram.runSort(new HeapSort())
// sort using linear search
sortProgram.runSort(new LinearSearch())
```

There was no interface but yet we did it. There could be a better and robust way but for now, this will suffice.

The thing here is to have it in my mind that for every sorting strategy we want to implement it must have a sort method where the sorting will be carried.

## Strategy Pattern: When To Use
Strategy Pattern should be used when you begin to notice recurring algorithms but in different variations. This way, you need to separate the algorithms into classes and feed them based on want in your program.

Next, if you notice recurring conditional statements around a related algorithm.

When most of your classes have related behaviors. It will be time to move them into classes.

## Advantages
* Separation of Concerns: Related behaviors and algorithms are separated into classes and strategies.
* Easy switching of strategies in runtime because you always program to interfaces.
* Elimination of bogus and conditional-infested code.
* Easy maintainability and refactoring.
* Choice of algorithms to use.

## Conclusion
Strategy Pattern is one of the many Design Patterns in software development. In this post, we saw many examples of how to use the SP and later on, we saw its benefits and drawbacks.

Remember, you don’t have to implement a design pattern as described. You have to thoroughly understand it and know when to apply it. And if you don’t understand it, no worry, keep referring to it again and again for insights. With time you’ll get the hang of it, and in the end, you will see the benefits.

Next, in our series, we will be looking into the **Template Method Design Pattern** so stay tuned :)

If you have any question regarding this or anything I should add, correct or remove, feel free to comment, email or [DM me](https://twitter.com/ngArchangel). Thanks for reading! 👏

## Credits
* [Design Patterns: Elements of Reusable Object-Oriented Software by Gamma, Helm, Johnson, & Vlissides, Addison Wesley, 1995](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612)
* [Pro TypeScript — Application-Scale JavaScript Development by Steve Fenton](https://www.amazon.com/Pro-TypeScript-Application-Scale-JavaScript-Development/dp/1484232488/ref=sr_1_1?s=books&ie=UTF8&qid=1543248511&sr=1-1&keywords=pro+typescript+steve+fenton)
* [The Strategy Pattern — Wikipedia](https://en.wikipedia.org/wiki/Strategy_pattern)
