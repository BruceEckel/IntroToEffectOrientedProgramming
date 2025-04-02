# Introduction to Effect-Oriented Programming (EOP)

In traditional programming, side effects (such as I/O operations, state mutations, or error throwing) are often interwoven with business logic, making programs *unpredictable* and harder to maintain.
Effect-Oriented Programming (EOP) treats **side effects** as first-class, manageable components of software.
EOP explicitly **separates side effects from pure logic**, providing a structured way to handle the unpredictable parts of a system ([Effect-Oriented Programming](https://developersummit.com/session/effect-oriented-programming#:~:text=A%20new%20paradigm%20called%20Effect,handling%20errors%2C%20applying%20timeouts%20and)) ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=,separately%20from%20the%20predictable%20ones)).
An *effect* represents an interaction with the outside world or a change in program state (for example, reading a file, modifying a global variable, or making a network request).
An **effect system** uses types to track these operations: a function's type signals the effects it may perform ([SoftwarePatternsLexicon.com/content/functional-programming/11/4/index.md at main · MasteryEducation/SoftwarePatternsLexicon.com · GitHub](https://github.com/MasteryEducation/SoftwarePatternsLexicon.com/blob/main/content/functional-programming/11/4/index.md#:~:text=Effect%20systems%20extend%20the%20type,the%20risk%20of%20unintended%20consequences)).
By partitioning the unpredictable parts of the code and managing them separately from the pure, deterministic parts, EOP makes programs more reliable and easier to reason about ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=,separately%20from%20the%20predictable%20ones)).

Under EOP, side effects become *explicit* in function signatures and program structure, rather than hidden in implementation.
The way effects are used or handled is enforced by the type checker ([SoftwarePatternsLexicon.com/content/functional-programming/11/4/index.md at main · MasteryEducation/SoftwarePatternsLexicon.com · GitHub](https://github.com/MasteryEducation/SoftwarePatternsLexicon.com/blob/main/content/functional-programming/11/4/index.md#:~:text=Effect%20systems%20extend%20the%20type,the%20risk%20of%20unintended%20consequences)) ([SoftwarePatternsLexicon.com/content/functional-programming/11/4/index.md at main · MasteryEducation/SoftwarePatternsLexicon.com · GitHub](https://github.com/MasteryEducation/SoftwarePatternsLexicon.com/blob/main/content/functional-programming/11/4/index.md#:~:text=,effecting%20operations)).
The result is code that is safer (fewer unexpected runtime errors), easier to test in isolation, and more adaptable to change.

## Benefits of Effect-Oriented Programming

Effect-Oriented Programming offers several concrete benefits that address long-standing pain points in software development.
By making side effects explicit and structured, EOP improves software **reliability**, simplifies **error handling**, enhances **testability**, supports safer **refactoring**, and clarifies reasoning about **asynchronous and concurrent** behavior.

### Reliability and Predictability of Software

By explicitly tracking side effects, an effect system can catch many issues at compile time--before the code ever runs.
For example, if a function can fail or perform I/O, those possibilities are reflected in its type, forcing the developer to acknowledge and handle them.
This compile-time enforcement reduces the risk of unhandled exceptions or hidden side effects surfacing as runtime bugs.
In other words, effect systems help ensure that side effects are **"managed safely and predictably,"** reducing unintended consequences ([SoftwarePatternsLexicon.com/content/functional-programming/11/4/index.md at main · MasteryEducation/SoftwarePatternsLexicon.com · GitHub](https://github.com/MasteryEducation/SoftwarePatternsLexicon.com/blob/main/content/functional-programming/11/4/index.md#:~:text=Effect%20systems%20extend%20the%20type,the%20risk%20of%20unintended%20consequences)) ([SoftwarePatternsLexicon.com/content/functional-programming/11/4/index.md at main · MasteryEducation/SoftwarePatternsLexicon.com · GitHub](https://github.com/MasteryEducation/SoftwarePatternsLexicon.com/blob/main/content/functional-programming/11/4/index.md#:~:text=effects,are%20managed%20safely%20and%20predictably)).
Code becomes more *predictable* in its behavior because any operation that might do something unusual (like disk access or network call) is declared up-front.

#### Explicit Separation of Pure and Impure Code

Pure functions (with no side effects) can be trusted to behave consistently given the same inputs, while effectful parts are isolated.
EOP enforces this separation, which means that the *"unpredictable parts of a system"* are partitioned away from the predictable ones ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=,separately%20from%20the%20predictable%20ones)).
This makes it easier to build *resilient systems*.
In fact, EOP has been touted as a powerful approach for building *reliable, composable, and maintainable software* by keeping side effects under control ([Effect-Oriented Programming](https://developersummit.com/session/effect-oriented-programming#:~:text=A%20new%20paradigm%20called%20Effect,handling%20errors%2C%20applying%20timeouts%20and)).
Compared to a traditional object-oriented approach—where any method might potentially modify state or throw an exception without warning—Effect-Oriented code is much more transparent about what it can and cannot do.
The increased transparency not only prevents many bugs but also makes it easier for developers to reason about correctness, since there are fewer "mystery" interactions happening behind the scenes.

### Simplified and Safer Error Handling

Conventional programming relies on exceptions or error codes that can be ignored or forgotten, leading to crashes or inconsistent states.
Effect-Oriented Programming instead treats errors as an explicit part of a function's effect.
This means functions can have *typed error effects*: the type system knows what kinds of errors might occur, and callers are forced to deal with those errors (for example, by handling a result type that could represent failure).
In practice, this is akin to returning errors as values (such as an `Either` or `Result` type) rather than throwing exceptions.
Modern effect systems provide **built-in error handling mechanisms**, allowing developers to **"handle errors as values, rather than exceptions."** This leads to safer code with no hidden failure paths ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=3)).
By making error possibilities part of the type, EOP ensures no error is forgotten--the compiler will complain if you fail to account for a potential failure case.

An additional benefit is the ability to implement robust error-handling policies like retries, timeouts, and fallback strategies in a uniform way.
Because effects are first-class, one can compose *error-handling logic* with the main computation.
For example, an effect system might let you wrap an operation with a retry policy without tangling the core logic with retry loops.
Indeed, EOP frameworks often come with *"superpowers for handling errors, applying timeouts and retries,"* all expressed declaratively as part of the effect system ([Effect-Oriented Programming](https://developersummit.com/session/effect-oriented-programming#:~:text=By%20explicitly%20separating%20side%20effects,testing%20unreliable%20components%20in%20isolation)).
This means you can apply these policies consistently across your codebase.
Compared to object-oriented code, which might use ad-hoc try/catch blocks scattered around, the Effect-Oriented approach centralizes and **standardizes error handling**, improving resilience.
Failures become easier to reason about since they are no longer out-of-band exceptions but anticipated outcomes.
Overall, treating errors as part of the effect type leads to more robust software: one study notes that making side effects (like exceptions) explicit allows many issues to be caught at compile time, *"reducing runtime errors."* ([SoftwarePatternsLexicon.com/content/functional-programming/11/4/index.md at main · MasteryEducation/SoftwarePatternsLexicon.com · GitHub](https://github.com/MasteryEducation/SoftwarePatternsLexicon.com/blob/main/content/functional-programming/11/4/index.md#:~:text=,effecting%20operations)).

### Modularity, Maintainability, and Refactorability

EOP encourages a highly **modular design**, which in turn improves maintainability and makes large codebases easier to refactor.
By separating pure logic from side-effectful operations, developers can organize code such that each piece has a single responsibility.
Pure functions handle core computations, while *effectful* functions deal with I/O or other interactions--and even those effectful parts are explicitly delineated.
This separation has several advantages.
First, it makes code more **readable and self-documenting**: when you see a function's type signature, you immediately know if it touches the network, disk, etc., or if it's free of side effects.
This clarity improves maintainability, as future maintainers (or your future self) can quickly understand the constraints of each component ([SoftwarePatternsLexicon.com/content/functional-programming/11/4/index.md at main · MasteryEducation/SoftwarePatternsLexicon.com · GitHub](https://github.com/MasteryEducation/SoftwarePatternsLexicon.com/blob/main/content/functional-programming/11/4/index.md#:~:text=Effect%20systems%20extend%20the%20type,the%20risk%20of%20unintended%20consequences)) ([SoftwarePatternsLexicon.com/content/functional-programming/11/4/index.md at main · MasteryEducation/SoftwarePatternsLexicon.com · GitHub](https://github.com/MasteryEducation/SoftwarePatternsLexicon.com/blob/main/content/functional-programming/11/4/index.md#:~:text=,effecting%20operations)).

Second, because the effect system enforces boundaries between pure and impure code, it becomes easier to **refactor** without introducing bugs.
If you attempt to move a piece of logic that performs an effect into a module that was supposed to be pure, the type checker will flag it--preventing a common class of errors.
Conversely, if you refactor an implementation and accidentally change its effects (say a function that used to only compute now also logs to a file), the changed effect type will ripple through compile-time checks, ensuring you update callers accordingly.
In a traditional paradigm, such a change might go unnoticed until runtime.
A strong effect typing thus gives you confidence during refactoring; the code is *harder to misuse*.
Developers have noted that in code without an organized effect system, *"effectful code is scattered everywhere"* and *"understanding, refactoring, and testing [such] a program…is a nightmare."* ([Effect-Oriented Programming - Programming Flix](https://doc.flix.dev/effect-oriented-programming.html#:~:text=Here%20every%20function%2C%20i.e.%20,scattered%20everywhere%20throughout%20the%20program)) EOP directly addresses this by localizing and controlling effects.

Modularity also means greater reuse.
If you have a generic effect for, say, "making an HTTP request," you can use it in many places and handle it in one place (for example, at program entry point or in a dedicated module).
This is superior to copying and pasting similar code (which often happens in OOP when each class manages its own low-level error handling or I/O).
As a concrete illustration, consider a scenario where you want a retry mechanism for unreliable operations.
In an object-oriented approach, you might implement retries inside each method that needs it, or use aspects or dependency injection to add it, both of which add complexity.
In an effect-oriented approach, you could implement a reusable **retry effect decorator** that can wrap any operation.
A developer recounting this approach in Python created a generic `Effect` type (a zero-argument function representing a deferred computation) and a reusable `retry` function to apply a retry policy to any effectful operation ([Be More Lazy, Become More Productive - DEV Community](https://dev.to/suned/be-more-lazy-become-more-productive-2cnb#:~:text=,Retrying)) ([Be More Lazy, Become More Productive - DEV Community](https://dev.to/suned/be-more-lazy-become-more-productive-2cnb#:~:text=We%27ll%20use%20this%20alias%20to,same%20backoff%20mechanism%20from%20before)).
By treating the operation as a value (an `Effect`), they separated the concerns of "what to do" from "how to retry it," achieving better reuse and adhering to the single-responsibility principle.
This kind of modular composition is a hallmark of EOP and greatly aids maintainability.

In summary, EOP's enforcement of effect boundaries yields code that is easier to **maintain and evolve** over time.
Changes in one part of the system (e.g., how an effect is implemented) have limited impact on pure logic elsewhere.
Moreover, the uniform approach to side effects across the codebase *"reduces cognitive load, making the codebase easier to understand and maintain."* ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=9)) ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=,common%20problems%2C%20speeding%20up%20development)) This consistency is invaluable in large systems.

### Enhanced Testability of Components

Making side effects explicit not only helps reliability, but also significantly improves **testability**.
In Effect-Oriented Programming, because functions declare their effects instead of performing them immediately and everywhere, it becomes possible to **substitute or simulate effects** during testing.
For example, if a function's type says it may perform a database access (as an effect), we can provide a fake implementation of that effect for testing purposes.
The function itself doesn't need to change--we simply run the test with a test-specific effect handler or mock, which returns controlled results instead of hitting a real database.
This ability to swap out effect implementations is analogous to dependency injection in OOP, but it's strongly enforced by the language's type system or effect runtime.
One benefit is that tests for pure logic can run without invoking any external systems, and tests for effectful logic can isolate those effects.
In an effect system, *"side-effecting code [can be tested] by simulating effects or running them in a controlled environment,"* which **increases test coverage and reliability** of the software ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=8)).

Another angle on testability is determinism.
Traditional code with uncontrolled side effects (e.g. random file I/O or time-dependent behavior) can be nondeterministic in tests.
Under EOP, since effects are isolated, you can often make tests deterministic by controlling when and how those effects happen.
For instance, you might replace a real clock effect with a simulated clock in tests, or a network call with a pre-defined response.
This makes it feasible to **unit test** scenarios that would otherwise be integration tests at best.
As noted in one conference talk description, EOP *"provides superpowers for testing unreliable components in isolation."* ([Effect-Oriented Programming](https://developersummit.com/session/effect-oriented-programming#:~:text=By%20explicitly%20separating%20side%20effects,testing%20unreliable%20components%20in%20isolation)) Because each effect can be handled separately, a component that depends on, say, user input can be tested by supplying a scripted input effect, verifying that the pure logic responds correctly.

In contrast, with a classic imperative approach, side effects are often tightly coupled to logic (for example, a function that reads user input and immediately processes it).
Testing such a function might require complicated setup or even manual steps.
With EOP, you could refactor that function to accept its input effect as a parameter or use an effect in its type, so that in production it reads from a real source, but in tests it uses a canned source.
The result is simpler, more **reliable automated tests**.
Empirical experience has shown that when effectful and pure parts are intermingled, *"testing a program written in this style is a nightmare."* ([Effect-Oriented Programming - Programming Flix](https://doc.flix.dev/effect-oriented-programming.html#:~:text=Understanding%2C%20refactoring%2C%20and%20testing%20a,this%20style%20is%20a%20nightmare)) EOP provides a disciplined alternative, yielding code that is crafted with testing in mind from the start.

### Better Reasoning for Asynchronous and Concurrent Programs

Effect-Oriented Programming also shines in the realm of **asynchronous and concurrent programming**.
Concurrency is notorious for being difficult to reason about and error-prone in imperative languages (with problems like race conditions, deadlocks, and leaked threads).
Many effect systems come with high-level concurrency primitives that make concurrent code both safer and easier to understand.
For example, an effect runtime might provide a concept of lightweight threads or fibers and ensure that resources are cleaned up and errors handled across these fibers automatically.
Because the effects of concurrent operations are tracked, the system can also enforce *structured concurrency* (making sure that child tasks complete or are canceled when expected).
From a developer's perspective, one interacts with concurrency through **declarative constructs** (like saying "run these two effects in parallel and then combine the results") rather than manually managing threads.

A major benefit here is that you can write asynchronous code that looks sequential, chaining effectful operations, and let the runtime handle scheduling.
The effect system can guarantee, for instance, that if one parallel task fails, the other is cancelled, and the error is propagated in a controlled way (all as part of the effect's semantics).
This leads to more **robust concurrent programs**.
Indeed, effect systems often provide *"high-level abstractions for concurrency and parallelism,"* allowing developers to focus on business logic instead of low-level thread management ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=4)).
Error handling and **cancellation** in concurrent scenarios are also simplified--many effect frameworks allow *safe interruption* of running tasks as an effect, ensuring that cleanup happens and no shared state is left in an inconsistent form ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=7)).
This is a stark improvement over traditional threads, where canceling a thread is fraught with danger.
As an example, an effect-oriented approach to concurrency might allow a function to have an effect type indicating it can fork new tasks; the system would then require the function's caller to handle the possibility of concurrent outcomes (like receiving multiple results or an error from a forked task).

Even for async (non-parallel) workflows, EOP provides clarity.
Consider JavaScript's `async/await`--it is not an effect system by itself, but it was introduced to make async I/O easier to follow by marking functions `async` and using `await` at call sites.
An effect system takes this idea further: it can mark not just async behavior but any side effect, and doesn't rely on developer discipline alone; it's usually baked into the type system.
The result is that reasoning about *when and how side effects occur* in an asynchronous program becomes tractable.
You can look at a type or effect annotation and know, for example, that "this function may perform network access but no other effects, and it might do so concurrently." This level of insight is typically impossible in a loosely-typed OOP system.
By leveraging effect types, developers gain a mental model of concurrent execution that is closer to the actual behavior, making complex asynchronous logic more understandable and dependable.

### Comparison with Traditional Paradigms

Effect-Oriented Programming can be seen as an evolution of ideas from functional programming applied to real-world software needs.
In traditional **imperative or object-oriented programming**, side effects are pervasive and unchecked: any method might change a global variable, perform I/O, or throw an exception without it being obvious from its signature.
Error handling in these paradigms is often *implicit* (exceptions bubble up until caught) and side effects are intermingled with logic, which can lead to tangled code.
For example, an object method might both compute a value *and* log it to a file as a side effect; the caller has no indication this logging occurs, and if you reuse the method in a context where file I/O isn't desired (say, in a GUI), you might get unwanted behavior.
Such hidden couplings make reasoning and maintaining imperative code challenging ([Effect Oriented Programming](https://effectorientedprogramming.com/#:~:text=Concerns%20like%20network%20communication%20or,reality%20of%20failures%20and%20inconsistency)) ([Effect Oriented Programming](https://effectorientedprogramming.com/#:~:text=Traditionally%2C%20we%27ve%20coped%20with%20Effects,to%20build%2C%20adapt%2C%20and%20maintain)).
As the authors of *Effect-Oriented Programming* note, without an effect discipline, *"the pristine world of algorithms devolves into the gory reality of failures and inconsistency,"* and we cope with these issues unwittingly, making programs hard to build and adapt ([Effect Oriented Programming](https://effectorientedprogramming.com/#:~:text=Concerns%20like%20network%20communication%20or,reality%20of%20failures%20and%20inconsistency)) ([Effect Oriented Programming](https://effectorientedprogramming.com/#:~:text=Traditionally%2C%20we%27ve%20coped%20with%20Effects,to%20build%2C%20adapt%2C%20and%20maintain)).

Object-oriented design offers some tools like abstraction and design patterns to manage side effects (for instance, using the Command pattern to represent operations, or dependency injection to supply different implementations for testing).
However, these are convention-based solutions and not enforced by the language's type system in most OOP languages.
By contrast, EOP bakes the management of side effects into the language's type or structural system, providing **guarantees**.
For instance, whereas Java's checked exceptions are a limited form of effect tracking (the compiler forces you to acknowledge certain exceptions), effect systems generalize this idea to *all kinds of effects*, not just errors ([[PDF] Side Effects - Computer Science: Indiana University](https://cs.indiana.edu/~sabry/papers/sideeffects.pdf#:~:text=One%20of%20the%20simplest%20and,that%20a%20computation%20may%20raise)).
Similarly, where an OOP approach might use interfaces and polymorphism to swap components (e.g., a fake database vs real database in tests), EOP uses effect types to achieve the same ends more directly and uniformly.

In imperative programming, there is also less distinction between calling a function that computes something and one that does I/O--you have to rely on documentation or naming to know the difference.
EOP makes this distinction explicit at the type level.
The net effect is that EOP can be viewed as a *more disciplined way to write programs that still perform side effects*, combining the predictability of functional programming with the practicality needed for I/O, user interaction, and state.
It doesn't replace object-orientation outright--you can use EOP in an OO language--but it changes how you structure your methods and classes.
Instead of letting every object manage its own side effects in an ad-hoc way, you funnel effects through well-defined channels (often ending at a central runtime or handler).
The result is software that is easier to reason about, as confirmed by experience: by **"explicitly separating side effects from the pure…parts of your code,"** EOP makes code more predictable and significantly eases maintenance and scaling of systems ([Effect-Oriented Programming](https://developersummit.com/session/effect-oriented-programming#:~:text=By%20explicitly%20separating%20side%20effects,testing%20unreliable%20components%20in%20isolation)) ([Effect-Oriented Programming](https://developersummit.com/session/effect-oriented-programming#:~:text=In%20this%20session%2C%20we%20will,side%20effects%20while%20preserving%20composability)).

In summary, while traditional paradigms leave the management of side effects to the programmer's discipline and design patterns, Effect-Oriented Programming provides a formal structure to capture, compose, and handle effects.
This leads to improvements in reliability, clarity, and adaptability that are difficult to achieve at scale with classical imperative or OOP techniques.

## Core Concepts of Effect-Oriented Programming

Having examined *why* Effect-Oriented Programming is beneficial, we now turn to *how* it works.
EOP is underpinned by several core concepts that enable its powerful handling of side effects.
Key ideas include treating effects as **first-class values**, using **typed effect signatures** (often encoding **typed errors**), **composing** complex operations from simpler effectful components, tracking **dependencies via types**, and describing side effects in a **declarative, composable** fashion.
These concepts are framework-agnostic--whether one is using Haskell's `IO` monad, Scala's ZIO, or a language with native effect handlers, the underlying principles are similar.
We will introduce each concept with examples to illustrate how they contribute to the EOP paradigm.

### Effects as First-Class Values

In EOP, an effect (an operation that may produce a side effect) is often represented as a **first-class value** in the program.
This means that instead of executing a side effect immediately, a function can *return an effect* (or an *effectful computation*) as a value, which can then be passed around, stored, or composed with other effects.
Treating effects as data in this way is transformative: it allows the program to **isolate** side effects and decide when and how to execute them.
As one developer succinctly put it, the core idea is *"thinking about effects as first-class values, so they can be isolated, tested, deferred until needed, etc."* ([Pure functions in JavaScript : r/javascript](https://www.reddit.com/r/javascript/comments/dp3yhh/pure_functions_in_javascript/#:~:text=As%20another%20user%20said%2C%20a,js)).
By turning what would be side-effecting actions into values, EOP effectively **takes the "side" out of "side effect"** ([Be More Lazy, Become More Productive - DEV Community](https://dev.to/suned/be-more-lazy-become-more-productive-2cnb#:~:text=This%20makes%20it%20hard%20to,effect)), integrating those actions into the normal flow of data and control.

A classic example comes from functional programming: in Haskell, an `IO` action is a value of type `IO<T>` which, when executed, produces a `T` (and may have side effects).
You can have a list of `IO` actions, pass an `IO` action as an argument, or choose not to execute an `IO` action at all.
Similarly, in an effect-oriented design in any language, you might represent an operation like "read from database" as an effect value (perhaps an object or closure) that encapsulates *what to do* but doesn't do it yet.
This enables powerful composition (as we'll discuss shortly) because you can construct bigger effects from smaller ones without performing them in the middle of composition.
The effect values are only *interpreted* or run by a dedicated part of the program (for example, the `main` function or a runtime engine that knows how to handle each effect).

Making effects first-class also facilitates **declarative programming** for side effects.
Instead of issuing commands step-by-step (imperatively), you build a description of the work to be done.
For instance, you might create an effect that represents "fetch data from URL and then transform it"--at this stage, it's just a description.
By not performing the fetch immediately, you could decide to run this effect later, run it multiple times, or not run it at all if certain conditions aren't met.
This separation of description from execution is a key aspect of EOP.
It has parallels to how we separate query preparation from execution in databases (SQL is declarative) or how UI frameworks separate the what (layout) from the when (render).
As one source notes, describing side effects in a purely functional (declarative) way means *"separating what to do from when and how to do it,"* which makes code more predictable and reasoned about ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=1,Effects)).
Overall, treating effects as first-class citizens is foundational: it unlocks the ability to compose and manipulate effectful computations with the same ease as pure data, but with type safety ensuring we don't accidentally run them in the wrong place.

### Typed Effects and Typed Errors

A cornerstone of effect-oriented systems is that **types carry information about effects**.
In practice, this often means that the type signature of a function will indicate not only the usual input and output types, but also the *effect(s)* it may perform.
We've touched on this already for error handling: an effect type might indicate that a function can fail with a certain error type.
More generally, an effect type can enumerate various kinds of side effects (for example, "this function may do `[DatabaseAccess, Logging]`").
By encoding effects in types, the language can enforce at compile time that code handles or propagates those effects correctly ([SoftwarePatternsLexicon.com/content/functional-programming/11/4/index.md at main · MasteryEducation/SoftwarePatternsLexicon.com · GitHub](https://github.com/MasteryEducation/SoftwarePatternsLexicon.com/blob/main/content/functional-programming/11/4/index.md#:~:text=Effect%20systems%20extend%20the%20type,the%20risk%20of%20unintended%20consequences)) ([SoftwarePatternsLexicon.com/content/functional-programming/11/4/index.md at main · MasteryEducation/SoftwarePatternsLexicon.com · GitHub](https://github.com/MasteryEducation/SoftwarePatternsLexicon.com/blob/main/content/functional-programming/11/4/index.md#:~:text=In%20this%20pseudocode%2C%20%60,a%20file%20is%20not%20allowed)).
This concept is known as a **type-and-effect system** in academic terms, and it extends the guarantees of the type system to cover side effects.

**Typed errors** are a notable application of this idea.
Instead of using untyped exceptions, an effect system might represent errors with an explicit error type.
For example, consider a function that reads a configuration file and may fail if the file is missing.
In an effect-oriented design, its type could be `Config readConfig(String path) throws FileNotFoundError` (using a pseudo-type syntax) or perhaps `Effect[FileNotFoundError, Config]`--the key is that `FileNotFoundError` is part of the type.
The caller of this function cannot ignore the possibility of `FileNotFoundError`; they must handle it (or themselves be an effectful function that allows that error to bubble up).
This is analogous to checked exceptions in Java, except generalized and made more composable.
As a reference, Java's checked exceptions were essentially an early form of effect typing (tracking in the method signature which exceptions might be thrown) ([[PDF] Side Effects - Computer Science: Indiana University](https://cs.indiana.edu/~sabry/papers/sideeffects.pdf#:~:text=One%20of%20the%20simplest%20and,that%20a%20computation%20may%20raise)), though Java's mechanism is limited to exceptions and often considered clunky.
Modern effect systems take the concept further--for instance, the Scala effect library ZIO attaches an error type to every effect, and the compiler ensures you don't accidentally forget to handle an error because it won't type-check if you do ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=3)).

Beyond errors, typed effects can include indicators for other side effects: network access, console I/O, state mutation, etc.
Some languages (like Flix or Koka) allow functions to be annotated with *effect labels* in their type.
For example, a Flix function might be declared as `def getUser(id: Int): User \ IO` to denote it does I/O, or more specifically `\ {Database}` if it talks to a database.
The compiler uses these annotations to ensure, for instance, that a function without the `IO` effect can't suddenly perform I/O unless its type is adjusted.
This provides a **soundness guarantee**: you know by looking at the type what side effects are *possible*.
As a concrete benefit, the effect type can *"prevent a function from being used in contexts where [that effect] is not allowed."* ([SoftwarePatternsLexicon.com/content/functional-programming/11/4/index.md at main · MasteryEducation/SoftwarePatternsLexicon.com · GitHub](https://github.com/MasteryEducation/SoftwarePatternsLexicon.com/blob/main/content/functional-programming/11/4/index.md#:~:text=)) For example, you can designate certain parts of your program as "pure" and the effect system will stop you from calling impure functions there.

It's worth noting that effect types often go hand-in-hand with **effect handlers** or runtime interpreters that handle the effects at the edge of the system.
For instance, if a function's type says it might perform a `Logging` effect, there will be a part of the program that provides an implementation for `Logging` (like writing to a log file or standard output).
The type ensures that the provision of this effect is accounted for--much like how a function requiring a dependency must be given that dependency.
This leads to the next concept, dependency tracking.

In summary, **typed effects** bring side effects into the domain of the compiler's checks.
This not only makes error handling safer (no more unhandled exceptions--they're just missing cases in a pattern match or an unwrapped error type), but also makes the *range* of a function's interactions explicit.
Developers gain a form of documentation and guarantee: if a function's type has no effect (often noted as `Pure` or just absence of an effect annotation), you can trust it has no side effects.
If it does have effects, you know exactly which ones and can plan accordingly.
This rigorous approach is what allows large effect-oriented codebases to remain reliable and refactorable--the types won't let you forget an effect or ignore a failure ([SoftwarePatternsLexicon.com/content/functional-programming/11/4/index.md at main · MasteryEducation/SoftwarePatternsLexicon.com · GitHub](https://github.com/MasteryEducation/SoftwarePatternsLexicon.com/blob/main/content/functional-programming/11/4/index.md#:~:text=,effecting%20operations)).

### Composing and Combining Effects

Because EOP treats effects as values and types, it enables **composability** of effectful operations in a principled way.
In traditional programming, combining two actions that both do side effects can be tricky: you have to decide an order, and if one fails you must manually handle how to undo or propagate, etc.
In effect-oriented programming, there are typically combinators or language constructs to **chain and combine effects** seamlessly, much like you would compose functions.
A simple composition is sequencing: doing effect A then effect B.
More complex compositions include running effects in parallel, racing two effects (whichever finishes first wins), or conditionally doing one effect based on the result of another.
Effect systems often provide a *monadic interface* (in functional terms) or similar, which means you can take the result of one effectful computation and feed it into the next, building a larger effect that represents "do A then B".
Importantly, when you compose effects, the resulting composed effect's type reflects the combination of effects.
For example, if A can do network I/O and B can do file I/O, the composed effect will indicate it may do both network and file I/O.

This structured composition is a huge improvement over ad-hoc imperative code because the effect system can **manage concerns like error propagation automatically**.
If effect A or effect B can fail, a good effect system will let you compose them in a way that if A fails, B is skipped and the whole composition is marked as failed--all with types ensuring you handle that situation.
Libraries and languages embodying EOP principles often boast that you can *"build complex operations from simpler ones using effect composability,"* which *"promotes code reuse and modularity."* ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=2)).
Indeed, instead of writing one monolithic function that does many things (and hides multiple side effects internally), you write small effectful pieces and then assemble them like building blocks.
This is analogous to how one would structure pure functions, but here those building blocks can include side effects.

Let's illustrate with a brief example scenario: Suppose you want to read some data from a web API, then save it to a file, then print a confirmation.
In an effect-oriented approach, you might have one effect for "fetch from API" (with a type indicating a network effect and potential error), another effect for "write file" (with a file I/O effect and potential error), and a third effect for "print line" (with a console effect).
You can compose them in sequence: first fetch, then write, then print.
The resulting effect is a single composite effect that carries the union of all those effects in its type (network + file + console, and perhaps an error type that is a combination of what each step might fail with).
Now you have a *description* of the whole workflow.
At some point, you provide actual handlers/runtimes for these (like actual network, actual filesystem, actual console) to *execute* it.
The composition could be done with a syntax like `fetchData(url).flatMap(data => writeFile(path, data)).flatMap(_ => printLine("Done"))` in a pseudo-functional style, or using `do` notation if available.
The key is that the **plumbing of passing results and propagating failures is handled by the effect system**, not by manually nested if-else or try-catch blocks.
This makes such code more declarative and significantly easier to refactor (e.g., you could reorder or reuse parts of the sequence without rewriting the control flow).

Furthermore, effect systems often support *parallel composition*: for instance, run two effects simultaneously and then combine their results.
In a well-designed system, even this has a simple combinator (say `zipPar` or similar) and the effect type will note the union of effects used by either branch.
This high-level composition of concurrency again frees the developer from managing threads and locking, while still making it clear what can happen.

The power of composition in EOP cannot be overstated--it is what allows building very complex behaviors (transactions, multi-step workflows, etc.) from small, testable units.
The phrase *"composable and declarative"* is often used: you declare what needs to happen and rely on the effect framework to compose the pieces correctly and efficiently ([Effect-Oriented Programming](https://developersummit.com/session/effect-oriented-programming#:~:text=In%20this%20session%2C%20we%20will,side%20effects%20while%20preserving%20composability)).
By contrast, in imperative programming, when you compose two operations you typically have to explicitly code how the control moves from one to the next and how errors are handled at each step.
EOP abstracts that control flow, letting you focus on the *essence* of the operations instead.
This leads to code that mirrors the high-level specification of the task, rather than being cluttered with boilerplate.

In summary, **effect composition** means that effectful computations enjoy the same modularity as pure functions.
You can pass them to higher-order functions, sequence them, choose among them, and build larger computations cleanly.
This composability is a primary reason developers find effect-oriented design scalable--you can grow the complexity of your program without a combinatorial explosion of complexity in handling interactions, because the effect system manages those interactions in a uniform way.
As one functional programming expert noted, this is arguably the top reason to use an effect system ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=2)), since it aids significantly in building scalable software.

### Type-Based Dependency Tracking

Another key concept of EOP is leveraging the type system to track **external dependencies** of code, which can be viewed as a special kind of effect.
Many effect systems allow you to encode in a function's type not just *what kinds* of side effects it can have (e.g. "does I/O"), but also *what contextual resources* it might need access to.
This often appears as an *environment* or *capability* requirement in the effect type.
For instance, a function might be written in a way that it doesn't perform logging directly, but its type indicates that "if you give me a Logging service, I can return a result"--effectively meaning it has a logging effect requirement.
In Scala's ZIO library (to use a concrete example for clarity), this is expressed with a type parameter `R` (environment) that a computation needs.
In languages with algebraic effects (like Eff or Flix), it might be represented as an effect label that must be handled by providing an implementation.
The idea is analogous to **dependency injection**, except done at the type level and often automatically.

For example, consider a function that needs to access a database.
In an object-oriented design, you might pass a database connection object into the function (or have it as a field).
In an effect-oriented design, you could instead have the function's type declare a `Database` effect in its signature.
The function itself doesn't know how to perform the database operation; it just declares "I need the ability to do database queries." At a higher level (the composition or the main program), that effect is satisfied by supplying an actual implementation (e.g., connecting to a real database or using an in-memory fake).
The benefit of this approach is that the compiler can ensure that **all dependencies are provided** and that no code performs an unauthorized action.
If a piece of code tries to access the database without declaring it, you get a compile error.
If it declares it, you must eventually handle it (provide the resource) or your program won't compile/run.
This is a **form of static dependency tracking**.

The Flix example we touched on earlier provides a good illustration: the guessing game program defined distinct effects `Guess`, `Secret`, and `Terminal` for different interactions (user input, random number generation, and console output) ([Effect-Oriented Programming - Programming Flix](https://doc.flix.dev/effect-oriented-programming.html#:~:text=Here%2C%20we%20have%20introduced%20three,algebraic%20effects)).
Each function listed exactly the effects it needed (for instance, the game loop needed `Guess` and `Terminal` but not `Secret`), making its dependencies explicit.
All the effects were then *handled in one place*--the `main` function provided concrete handlers for each (actual implementations using Java I/O) ([Effect-Oriented Programming - Programming Flix](https://doc.flix.dev/effect-oriented-programming.html#:~:text=def%20main%28%29%3A%20Unit%20%5C%20,resume%29)) ([Effect-Oriented Programming - Programming Flix](https://doc.flix.dev/effect-oriented-programming.html#:~:text=%7D%20with%20handler%20Secret%20,readLine)).
This demonstrates how EOP can achieve **inversion of control**: the high-level logic (game loop) is written abstractly against effects, and the actual side-effect implementation is plugged in later.
The types ensure that you haven't forgotten to supply a `Guess` or `Terminal` implementation.
The result was that *"the business logic is purely functional"* and *"where impurity is needed, it is precisely encapsulated by the use of effects and handlers."* ([Effect-Oriented Programming - Programming Flix](https://doc.flix.dev/effect-oriented-programming.html#:~:text=3.%20A%20,of%20printing%20to%20the%20console)).
In other words, the function could not do any I/O except through the capabilities it declared, and those were satisfied at the boundary.

Type-based dependency tracking goes beyond just I/O.
It can encapsulate things like configuration contexts, access to certain hardware, or even *non-functional requirements* like "this code may only run if the user is authenticated" (imagine an effect representing an authentication context).
While not all effect systems are this expressive, the general principle is to use types as a bookkeeping device for *what a piece of code needs from its environment*.
This leads to very **decoupled code**: your core logic doesn't hardcode global accesses or specific APIs; it just declares needs which can be fulfilled differently in different situations.
Testing is again a winner here, since you can give a test double for a dependency easily.
But even in production, this makes it easier to evolve the system (e.g., swap out one logging mechanism for another) because those dependencies are isolated.

In summary, EOP often employs types to achieve **dependency injection at compile time**.
If traditional DI frameworks are like runtime contracts ("if you don't set up the wiring, you get a null at runtime"), effect dependency tracking is a compile-time contract ("if you don't provide this effect, the code won't compile/run at all").
It results in clear, maintainable code where each component's external needs are transparent.
This aligns with the broader theme: making implicit assumptions explicit and checkable.
Through typed dependencies, effect-oriented codebases can enforce architecture constraints and prevent misconfiguration or misuse of APIs by construction.

### Declarative Description of Side Effects

Finally, one of the overarching themes of Effect-Oriented Programming is that it enables a more **declarative style** of handling side effects.
We touched on this under first-class effects and composition, but it's worth calling out explicitly.
In a declarative approach, you describe *what* you want to happen, and the underlying system (the effect runtime or framework) figures out *how and when* to make it happen, under the rules of the effect system.
This contrasts with the typical imperative style of writing out step-by-step instructions.
EOP, by its nature, encourages you to write **side-effecting logic in a declarative, composable manner** ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=1,Effects)).

For example, if you want to ensure that a certain cleanup happens whether or not an error occurred, in an imperative language you'd use `try/finally` or `try-with-resources`.
In an effect system, you might use a combinator that ensures resource safety (many effect frameworks provide something like a `bracket` or `acquireRelease` operation).
You declaratively wrap an effect with a resource acquisition and release pair, and the effect system guarantees the release part runs.
Another example is expressing *concurrency* declaratively: instead of spawning threads and managing them, you might declare that two tasks should run in parallel and let the system schedule them.
The effect runtime handles details like thread pooling, synchronization, cancellation, etc., guided by the high-level description you provided.
This is why effect systems can claim benefits like *"simplifies concurrent code, focusing on business logic over thread management."* ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=4))--the developer writes what needs to happen (e.g., "these tasks in parallel, then that task") and the library does the imperative work behind the scenes.

Declarative effect management also means side effects are **composable in the same way as data transformations**.
You can map over an effect (apply a pure function to its result), or filter, or fold a collection of effects, etc., all without executing side effects during the composition phase.
The actual effects occur only when the composite is executed.
This is similar to building a query plan in a database: you assemble what you want (join these, filter that, aggregate) and the database decides when to perform disk reads or other side effects.
In EOP, *the program itself becomes a higher-level specification* of behavior.
This not only improves clarity but also can unlock optimizations or safety guarantees.
For instance, an effect system might analyze that certain effects commute (can be reordered safely) or that an effect has no outcome on the result and can be eliminated--these are advanced topics, but they show the potential power of having effects as part of the language's semantic model.

In practice, what developers experience is that with EOP their code for handling things like timeouts, logging, or retries looks declarative.
You might write something like `withRetry(3, myEffect)` to indicate an effect should be retried up to 3 times; or `timeout(5.seconds, myEffect)` to indicate it should be canceled if it runs too long.
These read like high-level policies rather than low-level control flow, which makes the code self-explanatory.
Because such constructs are part of an effect system, they compose nicely (you could, for example, run two effects in parallel, each with its own retry policy, all inside a larger timeout).
This composability is a direct result of the declarative design--each effect description remains modular.

To sum up, the declarative nature of EOP means that developers focus on **describing side effects and their relationships** instead of orchestrating them manually.
The shift is similar to the shift from assembly to high-level programming: you tell the system *what* you want to achieve, not exactly *how* to twiddle the bits.
By raising the level of abstraction for side effects, EOP makes complex behavior easier to implement correctly.
It aligns with functional programming principles in which you try to minimize implicit, hidden actions and instead work with explicit representations that can be checked and manipulated safely.
This declarative philosophy underpins why EOP leads to code that is easier to reason about: when reading effect-oriented code, one can understand the flow of effects (and their potential failures, order, etc.) from the structure of the code and types, rather than needing to simulate mentally what the program does step by step.
This high-level clarity, backed by formal guarantees from the effect system, is what makes Effect-Oriented Programming a compelling paradigm for modern software development ([Effect-Oriented Programming](https://developersummit.com/session/effect-oriented-programming#:~:text=In%20this%20session%2C%20we%20will,side%20effects%20while%20preserving%20composability)) ([Effect-Oriented Programming](https://developersummit.com/session/effect-oriented-programming#:~:text=By%20explicitly%20separating%20side%20effects,testing%20unreliable%20components%20in%20isolation)).

## Conclusion

Effect-Oriented Programming introduces a disciplined approach to handling side effects in software, marrying the predictability of static typing with the practical need for performing I/O, handling errors, and managing state.
For intermediate developers, adopting EOP means learning to shift some responsibilities to the type system and runtime--specifically, letting them track and manage the side effects that used to be scattered implicitly through your code.
In return, you gain code that is more robust (fewer surprises at runtime), easier to test and maintain, and more modular in design.
We discussed how EOP improves reliability by catching effect misuses early, how it enforces explicit error handling and thereby prevents bugs, how it enables swapping out effect implementations for testing or different deployment scenarios, and how it brings order to concurrency and asynchronous workflows.
We also delved into the core concepts that make these benefits possible: treating effects as values, using types to encode what a computation might do (or require), composing effects cleanly, and writing code that declares *what* it intends to do with side effects rather than doing it immediately in-place.

While we remained framework-agnostic in this introduction, it is worth noting that many modern languages and libraries are embracing these ideas--from academic languages with built-in effect systems, to popular libraries in Scala, Kotlin, TypeScript, and beyond.
The principles cited here are backed by both research (e.g., type-and-effect systems from programming language theory ([[PDF] Side Effects - Computer Science: Indiana University](https://cs.indiana.edu/~sabry/papers/sideeffects.pdf#:~:text=One%20of%20the%20simplest%20and,that%20a%20computation%20may%20raise))) and real-world experience (industrial-strength libraries and developer testimonials ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=3)) ([Effect-Oriented Programming](https://developersummit.com/session/effect-oriented-programming#:~:text=By%20explicitly%20separating%20side%20effects,testing%20unreliable%20components%20in%20isolation))).
As software systems grow in complexity and demand greater reliability, Effect-Oriented Programming provides a path forward to manage that complexity.
By understanding and applying the concepts outlined, an intermediate developer can elevate their code to be more **robust, clear, and adaptable** in the face of the unpredictable--turning side effects from an obstacle into a well-managed aspect of program design.

**Sources:**

- Frasure, B., Eckel, B., & Ward, J. *Effect-Oriented Programming*. Leanpub, 2024. (Discusses controlling unpredictable elements via effect systems) ([Effect Oriented Programming](https://effectorientedprogramming.com/#:~:text=Concerns%20like%20network%20communication%20or,reality%20of%20failures%20and%20inconsistency)) ([Effect Oriented Programming](https://effectorientedprogramming.com/#:~:text=Traditionally%2C%20we%27ve%20coped%20with%20Effects,to%20build%2C%20adapt%2C%20and%20maintain)).  
- Ward, J. "Effect-Oriented Programming." *Developer Summit 2024*. (Talk abstract on reliability and testability benefits of EOP) ([Effect-Oriented Programming](https://developersummit.com/session/effect-oriented-programming#:~:text=A%20new%20paradigm%20called%20Effect,handling%20errors%2C%20applying%20timeouts%20and)) ([Effect-Oriented Programming](https://developersummit.com/session/effect-oriented-programming#:~:text=By%20explicitly%20separating%20side%20effects,testing%20unreliable%20components%20in%20isolation)).  
- Magnusson, A. et al. "Effect-Oriented Programming in Flix." *Flix Language Documentation*. (Demonstrates separating effects from logic for better refactoring and testing) ([Effect-Oriented Programming - Programming Flix](https://doc.flix.dev/effect-oriented-programming.html#:~:text=Here%20every%20function%2C%20i.e.%20,scattered%20everywhere%20throughout%20the%20program)) ([Effect-Oriented Programming - Programming Flix](https://doc.flix.dev/effect-oriented-programming.html#:~:text=3.%20A%20,of%20printing%20to%20the%20console)).  
- Alexander, A. *Functional Programming FAQ: Effect System Benefits.* 2024. (Enumerates advantages of effect systems like composability, error handling, concurrency) ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=2)) ([Functional Programming FAQ: What are the benefits of an Effect System, like ZIO? | alvinalexander.com](https://alvinalexander.com/scala/functional-programming-faq-what-are-benefits-of-effect-system-zio/#:~:text=3)).  
- MasteryEducation. "Effect Systems and Type-Level Programming." *Software Patterns Lexicon*, 2024. (Explains effect systems for explicit side effects and compile-time safety) ([SoftwarePatternsLexicon.com/content/functional-programming/11/4/index.md at main · MasteryEducation/SoftwarePatternsLexicon.com · GitHub](https://github.com/MasteryEducation/SoftwarePatternsLexicon.com/blob/main/content/functional-programming/11/4/index.md#:~:text=Effect%20systems%20extend%20the%20type,the%20risk%20of%20unintended%20consequences)) ([SoftwarePatternsLexicon.com/content/functional-programming/11/4/index.md at main · MasteryEducation/SoftwarePatternsLexicon.com · GitHub](https://github.com/MasteryEducation/SoftwarePatternsLexicon.com/blob/main/content/functional-programming/11/4/index.md#:~:text=,effecting%20operations)).  
- Knoldus Inc. *"ZIO Effects."* Medium, 2023. (Describes modeling side effects as first-class values for reliability and testability) ([Zio Effects | by Knoldus Inc. | Medium](https://medium.com/@knoldus/zio-effects-d0ca9b549a32#:~:text=Functional%20effects%20allow%20for%20the,safe%20and%20composable%20manner)) ([Zio Effects | by Knoldus Inc. | Medium](https://medium.com/@knoldus/zio-effects-d0ca9b549a32#:~:text=Functional%20effects%20in%20ZIO%20also,a%20controlled%20and%20predictable%20manner)).  
- Reddit discussion: *"Pure functions in JavaScript"* (user *ScientificBeastMode* on managing effects as values) ([Pure functions in JavaScript : r/javascript](https://www.reddit.com/r/javascript/comments/dp3yhh/pure_functions_in_javascript/#:~:text=As%20another%20user%20said%2C%20a,js)).  
- Dev.to article: Suned, *"Be More Lazy, Become More Productive,"* 2021. (Illustrates treating side-effects as first-class via a Python example) ([Be More Lazy, Become More Productive - DEV Community](https://dev.to/suned/be-more-lazy-become-more-productive-2cnb#:~:text=,Retrying)).
