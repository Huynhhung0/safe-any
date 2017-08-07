
# Safe any

## TypeScript any is evil

Type `any` was introduced in TypeScript for gradual typing. In particular to
 progressively type an existing JavaScript project.

In a pure TypeScript project, `any` is only used to bypass the compiler.
  By using `any` you tell to the compiler to keep away and to blindly trust you.

For instance the following code compiles with strict null checks:

```typescript
const x: any = null
const fullname = x.fullname
```

`Object | null | undefined` provides a type-safe replacement in most of the
 cases where `any` is involved.

However, this latter type does not allow to structurally test an object.

The following code is invalid:

```typescript
const x: Object | null | undefined = ...
if (x !== null && typeof x === "object") {
    if (typeof x.fullname === "string") {
        // error: Property 'fullname' does not exist
    }
}

```

A type-safe alternative could be:

```typescript
const x: Object | null | undefined = ...
if (x !== null && typeof x === "object") {
    const y: {fullname?: Object | null} = x as {fullname?: Object | null}

    if (typeof y.fullname === "string") {
        const fullname = y.fullname
    }
}

```

This is a simple test. Imagine a test over a complex object structure with
 nested objects... Futhermore, the test is error-prone. If you forget to
 null-check `x`, the compiler does not blame you since there is a
  type-assertion. Is that really better than `any`? I guess no.


## Compiler-compliant and type-safe structural checks

`SafeAny<T>` provides a way to defensively test if an object is valid, without
 any type-assertion or compiler tricks.

If you wish to test if an object `x` is structurally equivalent to an object of
 type `T`, then `x` must be typed as `SafeAny<T>`. You can assign everyting
 to `x`.

As an example assume that we receive a well-formed json string. The sender
 claims that the json string represents `Person`. The following sample enables
 to test its claim:

 ```typescript
type Person = {fullname: string, birthYear: number}

const x: SafeAny<Person> = JSON.parse(untrustedJson)
if (typeof x === "object" && x !== null &&
    typeof x.fullname === "string" && typeof x.birthYear === "number") {

    // x is a Person
}
```


## FAQ

### What is equivalent to `Object | null | undefined`?

`SafeAny<object>` is equivalent to `Object | null | undefined`.


### What about type-union and type-intersection?

`SafeAny<T1 | T2>` is equivalent to `SafeAny<T1 & T2>`.

