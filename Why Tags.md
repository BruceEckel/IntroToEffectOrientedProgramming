

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

let count = 1

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

// Class Version (findable by instanceof):
class RobotClass implements Robot {
  kind: "robot" = "robot"
  constructor(public name: string, private emits: string) {}
  action(): string {
    return this.emits
  }
}

const p1: Person = { kind: "person", name: "Alice", action: () => "talks" }
const p2: Person = { kind: "person", name: "Bob", action: () => "monologues" }
const r1: Robot = { kind: "robot", name: "C3PO", action: () => "informs" }
const r2 = new RobotClass("R2D2", "beeps")
const r3: Robot = { kind: "robot", name: "T800", action: () => "terminates" }

const mix: unknown[] = [
  p1,
  { kind: "robot", name: 11, action: () => 42 },
  p2,
  { kind: "person", name: "NotAPerson", action: 99 },
  { kind: "person", name: "BadReturn", action: () => 111 },
  r1,
  { kind: "robot", name: "K2SO", action: (x: string) => `${x}` },
  r2,
  { name: "WALL-E", action: () => "cleans" },
  r3
]

let count = 1

// instanceof still only works for classes:
for (const r of mix)
  if (r instanceof RobotClass)
    console.log(`${count++}: [instanceof] ${r.name} ${r.action()}`)

//  type guard for robot:
function isTaggedRobot(x: any): x is Robot {
  return x?.kind === "robot"
}

// Finds only Robot objects using "trusted discriminant":
for (const r of mix)
  if (isTaggedRobot(r))
    console.log(`${count++}: [Robot (trusted)] ${r.name} ${r.action()}`)

function isTaggedPerson(x: any): x is Person {
  return x?.kind === "person"
}

for (const p of mix)
  if (isTaggedPerson(p)) {
    const line = count++
    try {
      console.log(`${line}: [Person (trusted)] ${p.name} ${p.action()}`)
    } catch (err) {
      console.log(`${line}: [Person (trusted)]\n${err}:\n${JSON.stringify(p)}`)
    }
  }

//  Thorough type guard for person:
function isPerson(x: any): x is Person {
  return (
    typeof x === "object" &&
    x !== null &&
    x.kind === "person" &&
    typeof x.name === "string" &&
    typeof x.action === "function" &&
    x.action.length === 0 &&
    typeof x.action() === "string"
  )
}

// Finds only Person objects:
for (const p of mix)
  if (isPerson(p))
    console.log(`${count++}: [Person (thorough)] ${p.name} ${p.action()}`)
```

```console
[LOG]: "1: [instanceof] R2D2 beeps" 
[LOG]: "2: [Robot (trusted)] 11 42" 
[LOG]: "3: [Robot (trusted)] C3PO informs" 
[LOG]: "4: [Robot (trusted)] K2SO undefined" 
[LOG]: "5: [Robot (trusted)] R2D2 beeps" 
[LOG]: "6: [Robot (trusted)] T800 terminates" 
[LOG]: "7: [Person (trusted)] Alice talks" 
[LOG]: "8: [Person (trusted)] Bob monologues" 
[LOG]: "9: [Person (trusted)]
TypeError: p.action is not a function:
{"kind":"person","name":"NotAPerson","action":99}" 
[LOG]: "10: [Person (trusted)] BadReturn 111" 
[LOG]: "11: [Person (thorough)] Alice talks" 
[LOG]: "12: [Person (thorough)] Bob monologues" 
```