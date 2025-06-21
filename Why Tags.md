# Type Tags

Like JavaScript, TypeScript uses _structural typing_, which can be positively described as "flexible," "dynamic," or "permissive."
Any object that conforms to a desired shape is acceptable, which produces a broad interpretation of what "type" means.
For small and simple projects like web pages, this approach is often workable.
As projects get larger and more complex, the "anything goes" attitude towards typing eventually becomes intractable.
This is one of the dominant reasons for the acceptance of TypeScript, as it improves type checking.
Because TypeScript is not a dramatic departure from JavaScript, it is more readily adopted than a more radical approach such as Elm (which, like TypeScript, produces JavaScript).

The following example demonstrates some of the improvements provided by TypeScript, as well as limitations:

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

It's possible to create a runtime check to verify the shape of an object, as seen in `isRobot`.
We first check to ensure that `x` is an `object`, because it could also be a primitive type.
Because `null` also qualifies as an `object`, we must explicitly check for `null`.
After that we can check for the type of the `name` and `action` fields, and verify that `action` takes no arguments.
However, after all this work we haven't achieved anything more that TypeScript's type checker, and a `Person` still qualifies as a `Robot`.

Structural typing makes programming "easier" in the small; it allows you to be sloppy when you are writing simple programs or doing experimental coding.
If you are writing basic web pages this may seem liberating, 
although even with the smallest systems there is a cognitive load to knowing all the special cases in the language.
When systems get larger and more complex, structural typing becomes a heavy burden and we need to impose a stricter type system.

In TypeScript, a _type tag_ (also known as a _discriminant_) is usually a string literal because this provides the strongest type narrowing capabilities. 
Although the tag can be any literal type (including numbers, booleans, or even enums), discriminated unions work best with string literal tags.
The TypeScript compiler recognizes string literals more directly when narrowing union types with `switch` or `if`.

In the following, the type tag is named `kind`, but it can have any name.
Notice that the type of `kind` is the string literal `"robot"` or `"person"`:

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

class BigRobot implements RobotWithBigness {
  kind: "robot" = "robot"
  bigness: "very" = "very"
  constructor(public name: string, private emits: string) {}
  action = (): string => this.emits
}

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
  // Upcasting:
  yield new BigRobot("BB8", "rolls")
}

const robots = Array.from(robotGenerator())

// Only a constructed class object is findable by instanceof:
robots.forEach(r => {
  // Both checks required, no upcasting:
  if (r instanceof BigRobot || r instanceof RobotClass)
    log("instanceof:", r.name, r.action())
})

// Type guard based only on tag:
function isTaggedRobot(x: any): x is Robot {
  return x?.kind === "robot"
}

// Find Robot objects using "trusted discriminant":
for (const item of robots) 
  log(`isTaggedRobot(${Object.entries(item)}): [${isTaggedRobot(item)}]`)

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
  .forEach(robot => log("ExactRobot:", robot.name, robot.action()))
```

**Output:**

```console
[LOG]: "instanceof:",  "R2D2",  "beeps" 
[LOG]: "instanceof:",  "BB8",  "rolls" 
[LOG]: "isTaggedRobot(name,R2D2,emits,beeps,kind,robot,action,() => this.emits): [true]" 
[LOG]: "isTaggedRobot(kind,robot,name,C3PO,action,() => "informs"): [true]" 
[LOG]: "isTaggedRobot(action,() => act + " spoof",name,K2SO,kind,robot,extra,foo): [true]" 
[LOG]: "isTaggedRobot(name,BB8,emits,rolls,kind,robot,bigness,very,action,() => this.emits): [true]" 
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
[LOG]: "ExactRobot:",  "R2D2",  "beeps" 
[LOG]: "ExactRobot:",  "C3PO",  "informs"
```

The sole job of the `kind` type tag is to uniquely identify the type of the object, no matter how that object is created.
This uniqueness, however, requires programmer discipline across the entire project.
As we see in `type Spoof`, it's possible to accidentally produce a type that conforms to a tagged type,
although the particular combination of `kind` and `robot` does seem unlikely.

In `RobotClass` you can see that the

While tagging is a significant improvement atop structural typing, there are still holes in the system:
- Casts using `as` must be disallowed (as much as is practical).
- A type with a different name but conformant structure (`Spoof`) can still pass through type checks; tagging doesn't guarantee uniqueness as a nominative system does.
- (more)

One strategy for creating a tighter type system is to require that all objects be created via a class constructor,
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