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
Casting with `as` should be disallowed in your code, and if you encounter bugs in other code, search for `as`.

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

While tagging is a significant improvement atop structural typing, there are still holes in the system:
- Casts using `as` must be disallowed (as much as is practical).
- A type with a different name but conformant structure (`Spoof`) can still pass through type checks; tagging doesn't guarantee uniqueness as a nominative system does.

If you need completely deterministic type checking, you must create a type guard that validates _everything_, as seen in `isExactRobot`.
This checks for constructed types using `instanceof`, verifies `kind`, `name` and `action`, and also ensures there are no additional properties.

Another strategy for creating a tighter type system is to require that all objects be created via a class constructor,
in which case `instanceof` is a reliable way to determine the type.
It's possible to validate this at runtime (but not during type checking):

```ts
// constructed-by.ts

function constructedBy<T>(x: unknown, classConstructor: new (...args: any[]) => T): x is T {
  return Object.getPrototypeOf(x) === classConstructor.prototype
}

class Robot {
  constructor(public name: string) {}
}

console.log(constructedBy(new Robot("R2D2"), Robot)) // true
console.log(constructedBy({ name: "R2D2" }, Robot)) // false
```

`constructedBy` checks an object's prototype explicitly, providing a reliable runtime check.

This strategy would likely impose impractical constraints for most systems.
Another approach, if you think there is a type tag collision, is to change your type tags or add a second tag.