## The Global Symbol Registry and Branding

Although type tags solve the basic non-uniqueness problem of structural typing, you may observe that there's nothing preventing one library from using the same type tag as another.
To solve this, TypeScript provides a mechanism called *branding*. 
To understand it, we must first explore supporting mechanisms in JavaScript and TypeScript.

_Symbols_ serve as unique identifiers, ensuring strong typing and preventing accidental misuse, thereby enhancing type safety and clarity in your programs.
The _global symbol registry_ creates reusable symbols by key, enabling safe and distinct branding of types in TypeScript. 

### What is a Symbol?

A _symbol_ is considered a primitive type in JavaScript, like `number`, `string`, or `boolean`. 
This means it represents a fundamental, low-level value. 
However, unlike `number` or `string`, which represent common data, symbols are used specifically to create unique and unguessable identifiers. 
They are primarily used as object property keys and for advanced type-level patterns in TypeScript.

Symbols are used to create unique identifiers. 
Symbols are always unique, even if they have the same description:

```typescript
const sym1 = Symbol("identifier")
const sym2 = Symbol("identifier")

console.log(sym1 === sym2) // false
```

The description `"identifier"` helps with debugging but doesn't affect the symbol's uniqueness.

Although symbols are unique and opaque, they support a few well-defined operations:

* Create a symbol: using `Symbol(description)` or `Symbol.for(key)`
* Use as object keys: symbols can be used as property keys in objects
* Compare: symbols can be compared with `===` for identity
* Retrieve description: `sym.description` returns the debug label
* Global key lookup: `Symbol.keyFor(sym)` returns the key if created via `Symbol.for`
* Use with _computed property syntax_: symbols must be bracketed when used as keys (described later)

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
console.log(Symbol.keyFor(localSym)) // Undefined (not globally registered)
```

Using descriptive, namespaced keys helps avoid symbol collisions:

* Good: `"effect/Brand"`
* Poor: `"Brand"` (too generic)

The key (`"effect/Brand"`) is purely descriptive—it has no relationship to any filesystem path or URL.

No built-in JavaScript or TypeScript mechanism exists to list or inspect the global registry. The registry is intentionally opaque and secure by design. If you need a list, manually track symbols within your application.


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

### TypeScript and `unique symbol`

The `unique` keyword in TypeScript is used exclusively with `symbol` to define a `unique symbol` type. It is not used elsewhere in the language.

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


