

```ts
// RobotFinder.ts

type Person = { 
  name: string
  action(): string 
}

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

const mix = [
  { name: "Alice", action: () => "talks" } as Person,
  { name: 11, action: () => 42 },  // 'as' gives type error
  { name: "Bob", action: () => "monologues" } as Person,
  { name: "?", action: 99 },  // 'as' gives type error
  { name: "C3PO", action: () => "informs" } as Robot,
  { name: "K2SO", action: (x: string) => `${x}` } as Robot,
  new RobotClass("R2D2", "beeps")
]

// Only a constructed class object is findable by instanceof:
mix.forEach((item, index) => {
  if(item instanceof RobotClass)
    console.log(`${index}: [instanceof]`, item.name, item.action())
})

// Structural shape guard, works for all Robot shapes:
function isRobot(x: any): x is Robot {
  return (
    typeof x === "object" &&
    x !== null &&
    typeof x.name === "string" &&
    typeof x.action === "function" &&
    x.action.length === 0  // No arguments
  )
}

mix.forEach((item, index) => {
  if (isRobot(item))
    console.log(`${index}: [isRobot]`, item.name, item.action())
})
```

```console
[LOG]: "6: [instanceof]",  "R2D2",  "beeps" 
[LOG]: "0: [isRobot]",  "Alice",  "talks" 
[LOG]: "2: [isRobot]",  "Bob",  "monologues" 
[LOG]: "4: [isRobot]",  "C3PO",  "informs" 
[LOG]: "6: [isRobot]",  "R2D2",  "beeps" 
```

You cannot use `instanceof` with type aliases.
Only classes have runtime constructors so `r instanceof Robot` won't compile.

In TypeScript, a type tag (also known as a discriminant) is usually a string literal, and using a string literal provides the strongest type narrowing capabilities. 
Although the tag can be any literal type (including numbers, booleans, or even enums), discriminated unions work best with string literal tags.
The TypeScript compiler recognizes string literals more directly when narrowing union types with `switch` or `if`.

```ts
// TaggedRobotFinder.ts
type Person = {
  kind: "person"
  name: string
  action(): string
}

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

const mix: unknown[] = [
  new RobotClass("R2D2", "beeps"),
  { kind: "person", name: "Alice", action: () => "talks" } as Person,
  { kind: "robot", name: 11, action: () => 42 }, // as Robot fails
  { kind: "person", name: "Bob", action: () => "monologues" } as Person,
  { kind: "person", name: "NotAPerson", action: 99 }, // as Person fails
  { kind: "person", name: "BadReturn", action: () => 111 }, // as Person fails
  { name: "Morty", action: () => "aw geez" } as Person,
  { kind: "robot", name: "C3PO", action: () => "informs" } as Robot,
  { kind: "robot", name: "K2SO", action: (x: string) => `${x}` } as Robot,
  { name: "WALL-E", action: () => "cleans" } as Robot,
  { kind: "robot", name: "T800", action: () => "terminates" } as Robot
]

// instanceof still only works for classes:
mix.forEach((item, index) => {
  if (item instanceof RobotClass)
    console.log(`${index}: [instanceof]`, item.name, item.action())
})

// Robot type guard:
function isTaggedRobot(x: any): x is Robot {
  return x?.kind === "robot"
}

// Finds Robot objects using "trusted discriminant":
mix.forEach((item, index) => {
  if (isTaggedRobot(item))
    console.log(`${index}: [Robot (trusted)] `, item.name, item.action())
})

function isTaggedPerson(x: any): x is Person {
  return x?.kind === "person"
}

mix.forEach((item, index) => {
  if (isTaggedPerson(item)) {
    try {
      console.log(`${index}: [Person (trusted)]`, item.name, item.action())
    } catch (err) {
      console.log(`${index}: [Person (trusted)]\n${err}:\n${JSON.stringify(item)}`)
    }
  }
})

// Thorough type guard for person:
function isPerson(x: any): x is Person {
  return (
    typeof x === "object" &&
    x !== null &&
    x.kind === "person" &&
    typeof x.name === "string" &&
    typeof x.action === "function" &&
    typeof x.action() === "string"  &&
    x.action.length === 0
  )
}

mix.forEach((item, index) => {
  if (isPerson(item))
    console.log(`${index}: [Person (thorough)]`, item.name, item.action())
})
```

```console
[LOG]: "0: [instanceof]",  "R2D2",  "beeps" 
[LOG]: "0: [Robot (trusted)] ",  "R2D2",  "beeps" 
[LOG]: "2: [Robot (trusted)] ",  11,  42 
[LOG]: "7: [Robot (trusted)] ",  "C3PO",  "informs" 
[LOG]: "8: [Robot (trusted)] ",  "K2SO",  "undefined" 
[LOG]: "10: [Robot (trusted)] ",  "T800",  "terminates" 
[LOG]: "1: [Person (trusted)]",  "Alice",  "talks" 
[LOG]: "3: [Person (trusted)]",  "Bob",  "monologues" 
[LOG]: "4: [Person (trusted)]
TypeError: item.action is not a function:
{"kind":"person","name":"NotAPerson","action":99}" 
[LOG]: "5: [Person (trusted)]",  "BadReturn",  111 
[LOG]: "1: [Person (thorough)]",  "Alice",  "talks" 
[LOG]: "3: [Person (thorough)]",  "Bob",  "monologues" 
```