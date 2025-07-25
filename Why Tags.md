# Type Tags

Like JavaScript, TypeScript uses _structural typing_, which can be positively described as "flexible," "dynamic," or "permissive."
Because any object that conforms to a desired shape is acceptable, this produces a broad interpretation of what "type" means.
For small and simple projects like web pages, this approach is often workable.
As projects get larger and more complex, the "anything goes" attitude towards typing eventually becomes intractable.
This is one of the dominant reasons for the acceptance of TypeScript, as it improves type checking.
Because TypeScript is not a dramatic departure from JavaScript, it is more readily adopted than a more radical approach such as Elm (which, like TypeScript, produces JavaScript).

This example demonstrates some of the type improvements provided by TypeScript, as well as limitations:

```ts
// robot-finder.ts

type Robot  = { 
  name: string
  action(): string 
}

class RobotClass implements Robot {
  constructor(public name: string, private emits: string) {}
  action(): string {
    return this.emits
  }
}

type Person = { 
  name: string
  action(): string 
}

const p1: Person = { name: "Bob", action: () => "hi Bob"}
// Type annotation gives good errors
// const p2: Person = { name: "Bob"} // Error: action missing
// const p3: Person = { action: () => "hi Bob"} // Error: name missing
// const p4: Person = {} // Error: name, action missing
// const p5: Person = { name: "Bob", action: () => 11 } // Type error
// Don't use 'as' casting:
const p6: Person = { name: "Bob"} as Person
const p7: Person = { action: () => "hi Bob"} as Person
const p8: Person = {} as Person
// Although *sometimes* it catches errors:
// const p9: Person = { name: "Bob", action: () => 11 } as Person

const people: Array<Person> = [
  { name: "Person Alice", action: () => "talks" },
  { name: "Person Bob", action: () => "monologues" },
  // { name: "?", action: 99 },  // Type error
  // { name: 11, action: () => 42 },  // Type errors
  // { name: "Alf", action: (x: string) => `${x}` }, // Type error
  { name: "Robot C3PO", action: () => "informs" },
  { name: "Robot K2SO", action: () => "pilots" } as Robot,
  new RobotClass("Robot R2D2", "beeps")
]

const robots: Array<Robot> = [
  { name: "Robot C3PO", action: () => "informs" },
  new RobotClass("Robot R2D2", "beeps"),
  // { name: 11, action: () => 42 }, // Type error
  // { name: "?", action: 99 }, // Type error
  // { name: "K2SO", action: (s: string) => s }, // Type error
  { name: "Person Alice", action: () => "talks" },
  { name: "Person Bob", action: () => "monologues" } as Person,
]

// Only a constructed class object is findable by instanceof:
robots.forEach((item, index) => {
  if(item instanceof RobotClass)
    console.log(`${index}: [instanceof]`, item.name, item.action())
})

// Structural shape guard for all Robot shapes:
function isRobot(x: any): x is Robot {
  return (
    typeof x === "object" &&
    x !== null &&
    typeof x.name === "string" &&
    typeof x.action === "function" &&
    x.action.length === 0  // No arguments
  )
}

robots.forEach((item, index) => {
  if (isRobot(item))
    console.log(`${index}: [isRobot]`, item.name, item.action())
})
```

**Output:**

```console
[LOG]: "1: [instanceof]",  "Robot R2D2",  "beeps" 
[LOG]: "0: [isRobot]",  "Robot C3PO",  "informs" 
[LOG]: "1: [isRobot]",  "Robot R2D2",  "beeps" 
[LOG]: "2: [isRobot]",  "Person Alice",  "talks" 
[LOG]: "3: [isRobot]",  "Person Bob",  "monologues" 
```

The `type` and `interface` keywords are only available in TypeScript.
`type` is more recent, and more flexible; you virtually never need to use `interface`.

Note that although `Robot` and `RobotClass` produce the same "structural shape," `RobotClass` is different because it is a formal class with a constructor.
You cannot use `instanceof` with type aliases.
Only classes have runtime constructors so `r instanceof Robot` returns `false` unless `r` has been created by a constructor.

`Person` has the same shape as `Robot` so the two appear identical to the structural type system.
Definitions `p1` through `p9` show the many different ways you can attempt to define a `Person`, 
and how the type system prevents you from creating incorrect `Person` objects.

One large hole in the type system is the ability to cast using `as`.
With `as` you can basically say that any object is any other object; only in a few cases will you get type errors when using `as`.
Casting with `as` should be avoided in your code, and if you encounter bugs in other code, search for `as`.

The `people` array is specified to hold `Person`, which automatically prevents many type errors.
However, `Robot` objects are allowed because they have the same shape, so declaring `people` as `Array<Person>` is misleading.
Similarly, `robots` is not really an `Array<Robot>` because it can also hold `Person` objects.

It's possible to create a runtime check (a.k.a. _guard_) to verify the shape of an object, as seen in `isRobot`.
We first check to ensure that `x` is an `object`, because it could also be a primitive type.
Because `null` also qualifies as an `object`, we must explicitly check for `null`.
After that we check for the type of the `name` and `action` fields, and verify that `action` takes no arguments.
Finally, notice that the return value is a boolean, but the return type of `isRobot` is `x is Robot`.
This is a _type predicate_.
It says, "When this function returns true, you can safely treat `x` as having type `Robot` in the code that follows."
This way, `isRobot` provides _type narrowing_ and if this function returns `true`, the TypeScript compiler will treat `x` as a `Robot` from that point on.
The actual result of `isRobot` is just a Boolean; the type narrowing is a side effect picked up by the TypeScript compiler.

Unfortunately, after all the work writing `isRobot`, we haven't achieved anything more than TypeScript's type checker, and a `Person` still qualifies as a `Robot`.

Structural typing makes programming "easier" in the small; it allows you to be sloppy when you are writing simple programs or doing experimental coding.
If you are writing basic web pages, this may seem liberating.
However, even with the smallest systems there is a cognitive load to knowing all the special cases in the language.
When systems get larger and more complex, structural typing becomes a heavy burden and we need to impose a stricter type system.

In TypeScript, a _type tag_ (also known as a _discriminant_) is usually a string literal because this provides the strongest type narrowing capabilities. 
Although the tag can be any literal type (including numbers, booleans, or even enums), discriminated unions work best with string literal tags.
The TypeScript compiler recognizes string literals more directly when narrowing union types with `switch` or `if`.

In the following, the type tag is named `kind`, but it can have any name.
Here, the type of `kind` is the string literal `"robot"` or `"person"`:

```ts
// tagged-robot-finder.ts
const log = console.log.bind(console)

type Robot = {
  kind: "robot"
  name: string
  action(): string
}

class RobotClass implements Robot {
  kind: "robot" = "robot"
  constructor(public name: string, private emits: string) {}
  action = (): string => this.emits
}

type Person = {
  kind: "person"
  name: string
  action(): string
}

type Spoof = {
  extra: string
  action(): string
  name: string
  kind: "robot"
}

// Factory function:
function makeSpoof(name: string, act: string): Spoof {
  return {
    action: () => act + " spoof",
    name,
    kind: "robot",
    extra: "foo"
  }
}

// Extend a type with a second type tag:
type RobotWithBigness = Robot & {
  bigness: "very"
}

// Implement extended type:
class BigRobot implements RobotWithBigness {
  kind: "robot" = "robot"
  bigness: "very" = "very"
  constructor(public name: string, private emits: string) {}
  action = (): string => this.emits
}

// Class inheritance:
class LittleRobot extends RobotClass {}

const badRobots: Array<Robot> = [
  // All produce type errors:
  // { kind: "robot", action: () => "informs" }
  // { kind: "robot", name: "K2SO", action: (x: string) => `${x}` },
  // { kind: "robot", name: 11, action: () => 42 },
  // { kind: "person", name: "Alice", action: () => "talks" },
  // { name: "Morty", action: () => "aw geez" },
  // Casts succeed, bad Robots:
  {} as Robot,
  { kind: "robot", name: "Demolition" } as Robot,
  { kind: "robot", action: () => "cleans" } as Robot,
  // 
]

function* robotGenerator(): Generator<Robot, void, unknown> {
  yield new RobotClass("R2D2", "beeps")
  yield { kind: "robot", name: "C3PO", action: () => "informs" }
  yield makeSpoof("K2SO", "security")
  // No upcasting allowed for non-constructed objects:
  // yield { kind:"robot", name: "BB8", action: () => "rolls", bigness:"very" }
  // Upcasting works with a constructed object:
  yield new BigRobot("BB8", "rolls")
  yield new LittleRobot("InsectBot", "flies")
}

const robots = Array.from(robotGenerator())

// Only a constructed class object is findable by instanceof:
robots.forEach(r => {
  // Both checks required, no upcasting for type inheritance,
  // but class inheritance is covered:
  if (r instanceof RobotClass || r instanceof BigRobot)
    log("instanceof:", r.name, r.action())
})

function show(obj: Record<string, unknown>): string {
  return Object
    .entries(obj)
    .sort(([k1], [k2]) => k2.localeCompare(k1))  // reverse sort keys
    .map(([key, value]) => `${key}: ${value}`)
    .join(", ");
}

// Type guard based only on tag:
function isTaggedRobot(x: any): x is Robot {
  return x?.kind === "robot"
}

// Find Robot objects using trusted discriminant:
robots.forEach(r => {
  log(`TaggedRobot [${isTaggedRobot(r)}]: ${show(r)}`)
})
    

// Pattern match on discriminant:
function detect(x: Person | Robot) {
  switch (x.kind) {
    case "person":
      log(x, `detected Person ${x.name}`)
      break
    case "robot":
      log(x, `detected Robot ${x.name}`)
      break
    default:
      log(x, "did not detect Person or Robot")
  }
}

robots.forEach(detect)

// Thorough type guard:
function isExactRobot(value: unknown): value is Robot {
  if (value instanceof RobotClass) return true
  if (typeof value !== "object" || value === null) return false
  const obj = value as Record<string, unknown>
  if (
    obj.kind !== "robot" ||
    typeof obj.name !== "string" ||
    typeof obj.action !== "function"
  ) return false
  // No extra own properties:
  const ownKeys = Object.keys(obj)
  const allowedKeys = ["kind", "name", "action"]
  return (
    ownKeys.length === allowedKeys.length &&
    allowedKeys.every(key => ownKeys.includes(key))
  )
}

robots
  .filter(isExactRobot)
  .forEach(r => log("ExactRobot:", r.name, r.action()))
```

**Output:**

```console
[LOG]: "instanceof:",  "R2D2",  "beeps" 
[LOG]: "instanceof:",  "BB8",  "rolls" 
[LOG]: "instanceof:",  "InsectBot",  "flies" 
[LOG]: "TaggedRobot [true]: name: R2D2, kind: robot, emits: beeps, action: () => this.emits" 
[LOG]: "TaggedRobot [true]: name: C3PO, kind: robot, action: () => "informs"" 
[LOG]: "TaggedRobot [true]: name: K2SO, kind: robot, extra: foo, action: () => act + " spoof"" 
[LOG]: "TaggedRobot [true]: name: BB8, kind: robot, emits: rolls, bigness: very, action: () => this.emits" 
[LOG]: "TaggedRobot [true]: name: InsectBot, kind: robot, emits: flies, action: () => this.emits" 
[LOG]: RobotClass: {
  "name": "R2D2",
  "emits": "beeps",
  "kind": "robot"
},  "detected Robot R2D2" 
[LOG]: {
  "kind": "robot",
  "name": "C3PO"
},  "detected Robot C3PO" 
[LOG]: {
  "name": "K2SO",
  "kind": "robot",
  "extra": "foo"
},  "detected Robot K2SO" 
[LOG]: BigRobot: {
  "name": "BB8",
  "emits": "rolls",
  "kind": "robot",
  "bigness": "very"
},  "detected Robot BB8" 
[LOG]: LittleRobot: {
  "name": "InsectBot",
  "emits": "flies",
  "kind": "robot"
},  "detected Robot InsectBot" 
[LOG]: "ExactRobot:",  "R2D2",  "beeps" 
[LOG]: "ExactRobot:",  "C3PO",  "informs" 
[LOG]: "ExactRobot:",  "InsectBot",  "flies" 
```

The sole job of the `kind` type tag is to uniquely identify the type of the object, no matter how that object is created.
This uniqueness, however, requires programmer discipline across the entire project.
As we see in `type Spoof`, it's possible to accidentally produce a type that conforms to a tagged type,
although that particular combination of `kind` and `robot` seems unlikely.

In `RobotClass` you can see that the `kind` tag must both repeat its type declaration _and_ assign an initial value, which can only be `"robot"`.

`Person` has the same shape as `Robot` but it has a different `kind` tag.
`Spoof` also has the same shape as `Robot`, even though the keys have a completely different order and there's an `extra` key that's not in `Robot`.
`RobotWithBigness` shows how to extend a `type` using an `&`, and it adds a second type tag, `bigness`.
In `BigRobot` we see both type tags initialized.

The `badRobots` array shows how the type checker detects type errors, although this checking is effectively disabled using `as`.

`robotGenerator` is a generator function, as distinguished by the asterisk in `function*`.
It is used here primarily as an exercise in using generators; `const robots` uses `Array.from` to immediately produce all the robots in `robotGenerator`.

We cannot `yield` a `Robot` structure that has extra elements (look for `bigness:"very"`).
`RobotWithBigness` is extended from `Robot` so that structure represents an implementation of what appears to be a legitimate structural type.
However, to "upcast" this to a `Robot` would require trimming off the `bigness:"very"` field.
The only way to safely upcast an object is to create it with `new`, as seen with `BigRobot` and `LittleRobot`.

Using `instanceof` requires an exact check when dealing with types.
That is, extending `type RobotWithBigness` from `type Robot` does not establish an inheritance relationship between `RobotClass` and `BigRobot`.
Thus, we must check for both types in the expression `if (r instanceof RobotClass || r instanceof BigRobot)`.

The type guard `isTaggedRobot` can be quite simple, as it only needs to test if the `kind` field is `"robot"`.
Because `x` is `any`, it can also be `null` so we need to use the `?` when checking `kind`.

The type tag/discriminant allows for easy pattern matching on types, as we can just `switch` on `kind`.

If you need completely deterministic type checking, you must create a type guard that validates _everything_, as seen in `isExactRobot`.
This checks for constructed types using `instanceof`, verifies `kind`, `name` and `action`, and also ensures there are no additional properties.

While basic tagging is a significant improvement atop structural typing, there are still holes in the system:
- Casts using `as` must be disallowed (as much as is practical).
- A type with a different name but conformant structure (`Spoof`) can still pass through type checks; tagging doesn't guarantee uniqueness as a nominative system does.

The second point is significant, and there's a solution.

## The Global Symbol Registry and Branding

Although type tags solve the basic non-uniqueness problem of structural typing, you may observe that there's nothing preventing one library from using the same type tag as another.
To solve this, TypeScript provides a mechanism called *branding*.
To understand it, we must first explore supporting mechanisms in JavaScript and TypeScript.

_Symbols_ serve as unique identifiers, ensuring strong typing and preventing accidental misuse, thereby enhancing type safety and clarity in your programs.
The _global symbol registry_ creates reusable symbols by key, enabling safe and distinct branding of types in TypeScript.

### What is a Symbol?

A _symbol_ is a primitive type in JavaScript, like `number`, `string`, or `boolean`.
This means it represents a fundamental, low-level value.
However, unlike `number` or `string`, which represent common data, symbols are used specifically to create unique and unguessable identifiers.
They are primarily used as object property keys and for advanced type-level patterns in TypeScript.

Symbols create unique identifiers, even if they have the same description:

```typescript
const sym1 = Symbol("identifier")
const sym2 = Symbol("identifier")

console.log(sym1 === sym2) // false
```

The description `"identifier"` helps with debugging but doesn't affect the symbol's uniqueness.
Calling the `Symbol` constructor always creates a unique symbol, regardless of the `string` argument.

Although symbols are unique and opaque, they support a few well-defined operations:

* Create with `Symbol(description)` or `Symbol.for(key)`
* Can be used as property keys in objects
* Can be compared with `===` to determine identity
* `sym.description` returns the description label (`"identifier"` in the above example)
* `Symbol.keyFor(sym)` returns the key if created via `Symbol.for()`
* _computed property syntax_: symbols must be bracketed when used as keys (described later)

Symbols cannot be concatenated, serialized to JSON, or implicitly converted to strings,
which helps preserve their uniqueness and privacy in object structures.

### The Global Symbol Registry

The global symbol registry is built into JavaScript.
It allows you to create and reuse symbols across different parts of your application by using a common key:

```typescript
const globalSym1 = Symbol.for("mySharedSymbol")
const globalSym2 = Symbol.for("mySharedSymbol")

console.log(globalSym1 === globalSym2) // true
```

When you use `Symbol.for(key)`, the runtime:

* Looks up the key (`"mySharedSymbol"`) in a global registry.
* Returns the existing symbol if found, otherwise creates a new one.

* **Local Symbol (`Symbol()`)**:

    * Always creates a new, unique symbol.
    * Not globally accessible.

* **Global Symbol (`Symbol.for(key)`)**:

    * Shared across your entire application.
    * Reusable by the same key.

```typescript
const localSymbol = Symbol("id")
const globalSymbol = Symbol.for("id")

console.log(Symbol.for("id") === globalSymbol) // true
// false (each Symbol("id") call produces a unique symbol):
console.log(Symbol("id") === localSymbol) 
```

You can retrieve the key of a global symbol using `Symbol.keyFor()`:

```typescript
const sym = Symbol.for("app/symbol")
console.log(Symbol.keyFor(sym)) // "app/symbol"

const localSym = Symbol("app/symbol")
// Undefined (not globally registered):
console.log(Symbol.keyFor(localSym)) 
```

Use descriptive, namespaced keys to help avoid symbol collisions:

* Good: `"effect/Brand"`
* Poor: `"Brand"` (too generic)

The key (`"effect/Brand"`) is purely descriptive—it has no relationship to any filesystem path or URL.

No built-in JavaScript or TypeScript mechanism exists to list or inspect the global registry. 
The registry is intentionally opaque and secure by design. 
If you need a list, manually track symbols within your application.


### Computed Property Keys

Symbols can be used as object property keys. To do this, use *computed property syntax*:

```typescript
const sym = Symbol.for("uniqueKey")

const obj = {
  [sym]: "value"
}

console.log(obj[sym]) // "value"
```

Computed property syntax uses the value of a variable or expression as a property key in an object.
The syntax uses square brackets (`[]`) around the variable or expression.
Here, the symbol itself (`sym`) is evaluated as the property key, creating a unique, collision-free key.
The expression inside the brackets is looked up in the global symbol registry (in the case of `Symbol.for`) or evaluated directly if it's a local symbol,
and its value becomes the property key:

```typescript
const keyName = "dynamicKey"
const obj = {
  [keyName]: "dynamic value"
}

console.log(obj.dynamicKey) // "dynamic value"
```

### `unique symbol`

The `unique` keyword in TypeScript is used exclusively with `symbol` to define a `unique symbol` type. 
It is not used elsewhere in the language.

This special type (`unique symbol`) allows the symbol to be treated as a **literal type**, enabling precise branding and enforcing nominal typing. For example:

```typescript
const Foo: unique symbol = Symbol("foo")
type Bar = { [Foo]: string }
```

Here, `Foo` is not just of type `symbol`—it is a specific, unique symbol literal.
This allows the compiler to distinguish it from all other symbols, which is crucial for patterns like branding.

`unique symbol` is useful for creating types that are distinct and can't be mixed accidentally:

```typescript
const BrandId: unique symbol = Symbol.for("brandId")

type BrandedString = string & { [BrandId]: "BrandedString" }

function acceptBrandedString(value: BrandedString) {
  console.log(value)
}

const regularString = "hello"
// acceptBrandedString(regularString) // Error: Type 'string' is not assignable

acceptBrandedString(regularString as BrandedString) // OK (explicit cast)
```

### Using Branding to Produce Nominal Typing

Branding allows you to differentiate between identical primitive types like numbers or strings for different purposes, ensuring type safety:

```typescript
const BrandTypeId: unique symbol = Symbol.for("effect/Brand")

type ProductId = number & {
  readonly [BrandTypeId]: {
    readonly ProductId: "ProductId"
  }
}
```

* The `readonly` keyword makes the branded property immutable, meaning the brand key cannot be changed once it's set.
  This guarantees that the brand's identity remains stable and secure.
* The inner `readonly ProductId: "ProductId"` provides an additional, immutable identifier unique to `ProductId`, further solidifying its uniqueness.
* The `number &` syntax creates an **intersection type**, combining the primitive type `number` with additional branding metadata.
  This ensures `ProductId` is fundamentally a number but has extra information attached for type safety.

Usage:

```typescript
const id = 123
const productId = id as ProductId // Explicit cast

function getProduct(id: ProductId) {
  console.log("Fetching product", id)
}

// getProduct(id) // Error: 'number' is not assignable to 'ProductId'
getProduct(productId) // OK
```

`BrandTypeId` is a global symbol used as a unique brand identifier.
`ProductId` becomes a branded type distinct from plain numbers, preventing accidental misuse.


## Using Branding to Solve the Robot/Human Problem

Branding solves the problem of two distinct types, such as `Robot` and `Person`, 
being incorrectly treated as identical by the TypeScript compiler if they use the same type tags and have the same shape. 
By introducing unique, compiler-recognized markers, branding differentiates otherwise identical types.

Here's how branding distinguishes `Robot` from `Person`:

```typescript
const RobotBrand: unique symbol = Symbol.for("type/Robot")
const PersonBrand: unique symbol = Symbol.for("type/Person")

type Robot = {
  kind: "robot"
  name: string
  action(): string
  readonly [RobotBrand]: "Robot"
}

class RobotClass implements Robot {
  kind: "robot" = "robot"
  readonly [RobotBrand]: "Robot" = "Robot"
  constructor(public name: string, private emits: string) {}
  action = (): string => this.emits
}

type Person = {
  kind: "person"
  name: string
  action(): string
  readonly [PersonBrand]: "Person"
}

function createRobot(name: string, emits: string): Robot {
  return new RobotClass(name, emits)
}

function createPerson(name: string): Person {
  return {
    kind: "person",
    name,
    action: () => "speaks",
    [PersonBrand]: "Person"
  }
}

const robot = createRobot("R2D2", "beeps")
const person = createPerson("Alice")

function interact(entity: Robot | Person) {
  switch(entity.kind) {
    case "robot":
      console.log(`Robot named ${entity.name} ${entity.action()}`)
      break
    case "person":
      console.log(`Person named ${entity.name} ${entity.action()}`)
      break
  }
}

interact(robot)
interact(person)

// TypeScript prevents structural confusion:
// interact({ kind: "robot", name: "Spoof", action: () => "fake" }) // Error: Missing [RobotBrand]
```

* `RobotBrand` and `PersonBrand` are global symbols ensuring uniqueness.
* Each type includes a brand marker, uniquely differentiating them even if structurally identical otherwise.
* The `readonly` property ensures the branding cannot be changed or misused, thus preserving type safety.

Branding thus creates a nominal typing system within TypeScript’s structural typing framework, enforcing stricter type checks and significantly enhancing robustness, especially in larger and more complex projects.

### Are Type Tags Redundant with Branding?

In theory, branding makes type tags like `kind: "robot"` redundant because the brand already distinguishes the type. 
However, in practice, type tags and branding serve different roles and work best when used together:

| Feature                | Type Tag (`kind`)                               | Branding (`unique symbol`)                                          |
|------------------------|-------------------------------------------------|---------------------------------------------------------------------|
| **Purpose**            | Enables type narrowing via discriminated unions | Enforces nominal typing to distinguish structurally identical types |
| **Checked at runtime** | Yes (e.g., `if (x.kind === "robot")`)           | No — erased at runtime                                              |
| **Compiler behavior**  | Used in `switch`/`if` narrowing                 | Used to prevent unintended assignment or inference                  |
| **Developer clarity**  | Readable and obvious during debugging           | Hidden unless inspected with symbols                                |
| **Erasure in JS**      | Remains (string literal)                        | Symbol key is retained but usually ignored                          |
| **Tooling support**    | Supported by default in control-flow narrowing  | Invisible to tooling unless specifically handled                    |

Branding is invisible at runtime and only aids the compiler. 
This makes it unsuitable for runtime checks like pattern matching, where `kind` is still essential. 
On the other hand, branding enforces true nominal distinctions, which `kind` alone cannot.

You should use **both** branding and type tags, so that:

* Branding provides **compile-time guarantees** and prevents mistaken assignments.
* Type tags provide **runtime discriminants** to make control flow and debugging clearer.

Together, they offer strong guarantees and clear semantics both for the compiler and for the human reader.
However, in many real-world scenarios you can choose the simpler solution between:

- Different type tags (like `"user_id"` vs `"product_id"`) often solve the problem more simply than branding
- Branding alone works well for primitive wrapper types
- Type tags alone work perfectly for discriminated unions with different shapes

The "use both" approach is mainly needed when you're constrained by existing APIs or need both runtime checks and compile-time nominal typing guarantees.
