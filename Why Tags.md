

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

// Class Version (findable by instanceof):
class RobotClass implements Robot {
  constructor(public name: string, private emits: string) {}
  action(): string {
    return this.emits
  }
}

const p1: Person = { name: "Alice", action: () => "talks" }
const p2: Person = { name: "Bob", action: () => "monologues" }
const r1: Robot  = { name: "C3PO",  action: () => "informs" }
const r2 = new RobotClass("R2D2", "beeps")

const mix = [
  p1, 
  { name: 11, action: () => 42 }, 
  p2, 
  { name: "?", action: 99 }, 
  r1, 
  { name: "K2SO", action: (x: string) => `${x}`}, 
  r2
]

// instanceof only works for class objects,
// only finds RobotClass:
for (const ra of mix)
  if(ra instanceof RobotClass)
    console.log("instanceof:", ra.name, ra.action())

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

for(const rb of mix)
  if(isRobot(rb))
    console.log("shape:", rb.name, rb.action())
```

You cannot use instanceof with type aliases — only classes have runtime constructors. So `r instanceof Robot` won't compile.

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

// instanceof still only works for classes:
for (const ra of mix)
  if (ra instanceof RobotClass)
    console.log("instanceof RobotClass", ra.name, ra.action())

//  type guard for robot:
function isTaggedRobot(x: any): x is Robot {
  return x?.kind === "robot"
}


// Finds only Robot objects using "trusted discriminant":
for (const rt of mix)
  if (isTaggedRobot(rt))
    console.log("[Robot (trusted)]", rt.name, rt.action())


function isTaggedPerson(x: any): x is Person {
  return x?.kind === "person"
}

for (const p of mix)
  if (isTaggedPerson(p)) 
    try { 
      console.log("[Person (trusted)]", p.name, p.action()) 
    } catch (err) { 
      console.log("[Person (trusted)]", err, p) 
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
    console.log("[Person (thorough)]", p.name, p.action())
```