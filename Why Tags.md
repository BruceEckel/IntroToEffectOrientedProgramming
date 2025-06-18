

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
// Don't use 'as':
const p6: Person = { name: "Bob"} as Person
const p7: Person = { action: () => "hi Bob"} as Person
const p8: Person = {} as Person
// Although *sometimes* it catches errors:
// const p9: Person = { name: "Bob", action: () => 11 } as Person

const people: Array<Person> = [
  { name: "Alice", action: () => "talks" },
  { name: "Bob", action: () => "monologues" },
  // { name: "?", action: 99 },  // Type error
  // { name: 11, action: () => 42 },  // Type errors
  // { name: "K2SO", action: (x: string) => `${x}` }, // Type error
  { name: "C3PO", action: () => "informs" },
  { name: "K2SO", action: () => "pilots" } as Robot,
  new RobotClass("R2D2", "beeps")
]

const robots: Array<Robot> = [
  { name: "C3PO", action: () => "informs" },
  new RobotClass("R2D2", "beeps"),
  // { name: 11, action: () => 42 }, // Type error
  // { name: "?", action: 99 }, // Type error
  // { name: "K2SO", action: (s: string) => s }, // Type error
  { name: "Alice", action: () => "talks" } as Person,
  { name: "Bob", action: () => "monologues" },
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

```console
[LOG]: "1: [instanceof]",  "R2D2",  "beeps" 
[LOG]: "0: [isRobot]",  "C3PO",  "informs" 
[LOG]: "1: [isRobot]",  "R2D2",  "beeps" 
[LOG]: "2: [isRobot]",  "Alice",  "talks" 
[LOG]: "3: [isRobot]",  "Bob",  "monologues" 
```

You cannot use `instanceof` with type aliases.
Only classes have runtime constructors so `r instanceof Robot` won't compile.

Structural typing makes programming "easier" in the small; basically it allows you to be sloppy when you are writing simple programs or doing experimental coding.
If you are writing simple web pages this may seem liberating, 
although even with the smallest systems there is a cognitive load to knowing all the special cases in the language.
When systems get larger and more complex, structural typing becomes a heavy burden and we need to impose a stricter type system.

In TypeScript, a type tag (also known as a discriminant) is usually a string literal because this provides the strongest type narrowing capabilities. 
Although the tag can be any literal type (including numbers, booleans, or even enums), discriminated unions work best with string literal tags.
The TypeScript compiler recognizes string literals more directly when narrowing union types with `switch` or `if`.

In the following, the type tag is `kind` (it can have any name).
Notice that the type of `kind` is the string literal `"robot"` or `"person"`:

```ts
// tagged-robot-finder.ts

type Robot = {
  kind: "robot"
  name: string
  action(): string
}

class RobotClass implements Robot {
  kind: "robot" = "robot"
  constructor(public name: string, private emits: string) {}
  action(): string {
    return this.emits
  }
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

const robots: Array<Robot> = [
  new RobotClass("R2D2", "beeps"),
  { kind: "robot", name: "C3PO", action: () => "informs" },
  makeSpoof("K2SO", "security"), // Identical shape
  // All produce type errors:
  // { kind: "robot", action: () => "informs" }
  // { kind: "robot", name: "K2SO", action: (x: string) => `${x}` },
  // { kind: "robot", name: 11, action: () => 42 },
  // { kind: "person", name: "Alice", action: () => "talks" },
  // { name: "Morty", action: () => "aw geez" },
  // Casts produce bad Robots:
  {} as Robot,
  { kind: "robot", name: "Demolition" } as Robot,
  { kind: "robot", action: () => "cleans" } as Robot,
  // 
]


// Only a constructed class object is findable by instanceof:
robots.forEach((item, index) => {
  if (item instanceof RobotClass)
    console.log(`${index}: [instanceof]`, item.name, item.action())
})

// Robot type guard:
function isTaggedRobot(x: any): x is Robot {
  return x?.kind === "robot"
}

// Finds Robot objects using "trusted discriminant":
robots.forEach((item, index) => {
  if (isTaggedRobot(item))
    try {
      console.log(`${index}: [Robot (tagged)] `, item.name, item.action())
    } catch(err) {
      console.log(`${index}: [Robot (tagged)]\n${err}:\n${JSON.stringify(item)}`)
    }
})

// Thorough type guard:
function isRobot(x: any): x is Robot {
  return (
    typeof x === "object" &&
    x !== null &&
    x.kind === "robot" &&
    typeof x.name === "string" &&
    typeof x.action === "function" &&
    typeof x.action() === "string"  &&
    x.action.length === 0
  )
}

robots.forEach((item, index) => {
  if (isRobot(item))
    console.log(`${index}: [Robot (thorough)]`, item.name, item.action())
})

// Pattern match on discriminant:
function detect(x: Person | Robot) {
  switch (x.kind) {
    case "person":
      console.log(`Detected Person ${x.name}`)
    case "robot":
      console.log(`Detected Robot ${x.name}`)
  }
}

for (const robot of robots) {
  console.log("testing", robot)
  detect(robot)
}
```

```console
[LOG]: "0: [instanceof]",  "R2D2",  "beeps" 
[LOG]: "0: [Robot (tagged)] ",  "R2D2",  "beeps" 
[LOG]: "1: [Robot (tagged)] ",  "C3PO",  "informs" 
[LOG]: "2: [Robot (tagged)] ",  "K2SO",  "security spoof" 
[LOG]: "4: [Robot (tagged)]
TypeError: item.action is not a function:
{"kind":"robot","name":"Demolition"}" 
[LOG]: "5: [Robot (tagged)] ",  undefined,  "cleans" 
[LOG]: "0: [Robot (thorough)]",  "R2D2",  "beeps" 
[LOG]: "1: [Robot (thorough)]",  "C3PO",  "informs" 
[LOG]: "2: [Robot (thorough)]",  "K2SO",  "security spoof" 
[LOG]: "testing",  RobotClass: {
  "name": "R2D2",
  "emits": "beeps",
  "kind": "robot"
} 
[LOG]: "Detected Robot R2D2" 
[LOG]: "testing",  {
  "kind": "robot",
  "name": "C3PO"
} 
[LOG]: "Detected Robot C3PO" 
[LOG]: "testing",  {
  "name": "K2SO",
  "kind": "robot",
  "extra": "foo"
} 
[LOG]: "Detected Robot K2SO" 
[LOG]: "testing",  {} 
[LOG]: "testing",  {
  "kind": "robot",
  "name": "Demolition"
} 
[LOG]: "Detected Robot Demolition" 
[LOG]: "testing",  {
  "kind": "robot"
} 
[LOG]: "Detected Robot undefined" 
```

While tagging is a significant improvement atop structural typing, there are still holes in the system:
- Casts using `as` must be disallowed
- A type with a different name but conformant structure (`Spoof`) can still pass through type checks; tagging doesn't guarantee uniqueness as a nominative system does.
- (more)

One strategy for creating a tighter type system would be to require that all objects be created via a class constructor,
in which case `instanceof` would be a reliable way to determine the type.
It's possible to validate this at runtime (but not during type checking):

```ts
// constructed-by.ts

function constructedBy<T>(x: unknown, classConstructor: new (...args: any[]) => T): x is T {
  return Object.getPrototypeOf(x) === classConstructor.prototype
}

class Robot {
  constructor(public name: string) {}
}

const a = new Robot("R2D2")
const b = { name: "R2D2" }  // Structurally identical

console.log(constructedBy(a, Robot)) // true
console.log(constructedBy(b, Robot)) // false
```

However, this would likely impose impractical constraints for most systems.