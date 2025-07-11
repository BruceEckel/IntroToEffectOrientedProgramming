### Computed Property Keys

TypeScript allows you to define object property keys using expressions, rather than fixed names. 
These are called **computed property keys**, and they are written inside square brackets `[]` in type or object definitions.

There are two major ways computed keys appear in branding:

1.  **Symbol-based computed keys:**

    ```ts
    readonly [BrandTypeId]: { ... }
    ```
    
    This creates a single property whose key is the value of the symbol `BrandTypeId`. 
    In this case, the key is not a string like "brand"--it's a symbol, which guarantees uniqueness and non-collision.
    If you pass in a string, it will produce an identifier with that string: 
    
    ```ts
    readonly ["Dog"]: { ... } // Identifier becomes Dog 
    ```

2.  **Mapped types using computed keys:**

    ```ts
    readonly [id in ID]: ID
    ```
    
    This is part of a _mapped type_, which iterates over a union type `ID`. For each member of the union, it creates a distinct property where the key is the value itself:
     
    ```ts
    type Keys = "a" | "b"

    type T = {
      [K in Keys]: number
    }
    ```
    
    Expands to:

    ```ts
    type T = {
      a: number
      b: number
    }
    ```
    
In a generic like this:

```ts
interface Brand<ID extends string | symbol> {
  readonly [BrandTypeId]: {
    readonly [id in ID]: ID
  }
}
```

When you call it with a single type argument `ID`, such as `Brand<"typeID">`, 
`readonly [id in ID]: ID` "iterates over" each of the types allowed by the generic (`string | symbol`) and produces a
key-value pair for each match. There's only one match ing this case, so it produces:

```ts
interface Brand<"typeID"> {
  readonly BrandTypeId: {
    readonly typeID: "typeID"
  }
}
```

In branding, this is intended for use with a single string or symbol literal to generate a unique shape.

Computed property keys are essential to branding in TypeScript because they allow the compiler to distinguish types using structural uniqueness—either by injecting symbol-based keys, or by generating objects with unique string keys tied to a specific brand identity.

The branding construct acts like a _type-level watermark_:

```ts
readonly [BrandTypeId]: {
  readonly ProductId: "ProductId"
}
```

The `BrandTypeId` symbol ensures the property is hidden and cannot conflict with other keys, 
while the nested `ProductId` string literal makes the brand uniquely identifiable. 
This combination gives `ProductId` a structural fingerprint that distinguishes it from all other types--even if they also extend `number`.

The branding watermark doesn't exist at runtime, but the TypeScript compiler sees it and treats the type as nominally unique.
If you cast a number to `ProductId` using `as`, the result is still just a plain number; calling `(brandedId as any)[BrandTypeId]` will return `undefined`. 

This inner string-literal field (`readonly ProductId: "ProductId"`) closely resembles a type tag. 
While it is never checked at runtime, it serves a similar purpose: providing a unique structural identity that the compiler can recognize. 
Think of it as a hidden tag nested inside a watermark, which helps encode nominal identity without polluting the surface shape of the type.
