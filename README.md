# type-systems-showdown
A wide ranging comparison-by-example of the typesystem features in various FP languages. Initially Purescript and Typescript.

## Untagged unions and friends

Purescript doesn't support untagged unions. But it can do something similar with a [library](https://github.com/jvliwanag/purescript-untagged-union). TODO: Comapre and contrast with Typescript.

Typescript
```typescript
type t1 = 1 | 2 | 3

type t2 = string | number

type t3 = t1 | t2
```

In all these cases there is no boxing.

Refinement of the type is done with [Flow sensitive typing](https://en.wikipedia.org/wiki/Flow-sensitive_typing)

E.g. -
Typescript
```typescript
// t :: t2
if(typeof t == "number") {
  // Here the type of t is refined to number
}
```

This can also work for user defined types with [User defined type guards](https://www.typescriptlang.org/docs/handbook/advanced-types.html#user-defined-type-guards). These however are completely unsafe and the compiler does not check if your type check is correct. This is effectively unsafeCoerce but localised to the type check function.

e.g. -
Typescript
```typescript
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

// Both calls to 'swim' and 'fly' are now okay.
let pet = getSmallPet();

if (isFish(pet)) {
  pet.swim();
} else {
  pet.fly();
}
```

*Structural* type narrowing is possible with `in` operator to check for fields -

Typescript -
```typescript
function move(pet: Fish | Bird) {
  if ("swim" in pet) {
    return pet.swim();
  }
  return pet.fly();
}
```

## Tagged Unions

Purescript
```purescript
data Shape = Square Number | Rectangle Number Number | Circle Number Number
area :: Shape -> Number
area s = case s of
  Square size -> size * size
  Rectangle width height -> width * height
  Circle radius -> Math.PI * radius * radius
```

Typescript requires defining records with a literal discriminant property for each variant.
```typescript
interface Square {
    kind: "square";
    size: number;
}

interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}

interface Circle {
    kind: "circle";
    radius: number;
}

type Shape = Square | Rectangle | Circle;

function area(s: Shape) {
    // In the following switch statement, the type of s is narrowed in each case clause
    // according to the value of the discriminant property, thus allowing the other properties
    // of that variant to be accessed without a type assertion.
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.width * s.height;
        case "circle": return Math.PI * s.radius * s.radius;
    }
}
```

Typescript's implementation is clearly clunkier, but it does allow discriminating on multiple properties, as well as discriminating on a subset of the variants.

Typescript
```typescript
interface FailState {
    error: "1";
    msg: string;
}

interface LoadingState {
    error: "0";
}

interface SuccessState {
    error: "0";
    value: number;
}

type State = FailState | LoadingState | SuccessState;

function processState(state : State) {
    switch (state.error) {
        case "0": {
            console.log("We did not fail!");
            break;
        }
        case "1": {
            console.log("We failed with ", state.msg);
            break;
        }
    }
}

processState({ msg: "unknown error", error: "1" });
```

## Variants

Typescript supports Variants natively with untagged unions as shown in the previous examples.
Purescript can have variants with the [Purescript-variant](https://github.com/natefaubion/purescript-variant) library.

**TODO: Elaborate.**

Challenge - declare a type which is similar to a previously defined type, but excludes a variant branch.

More specifically, given a type -

```purescript
data T = A | B | C Int
```

Now define a type function to generate a type U which is equivalent to a type -

```purescript
data U = B | C Int
```

In Typescript you can define the second type in terms of the first like this -
```typescript
type U = Exclude<T, 'a'>
const f = (t: T): U => t === 'A' ? 'B' : t
```

[Try it online!](https://www.typescriptlang.org/play?#code/C4TwDgpgBAKlC8UDkBBJUA+yBC6sG0kBhJAGigDsBXAWwCMIAnAXQChXRIoBVBKAUQAeAYwA2VACYQAPDHKokAPnbCA9hQDOwKADM+ACmAAuWAEoTveIqjb4d5GigB+HOhPAgA)

In Purescript we can denote the general relationship between any two data variants as long as the first has the field `A` and the second has the corresponding field `B` -
```purescript
_A = (SProxy :: _ “A”)
_B = (SProxy :: _ “B”)
f :: forall r. Variant (A::String | r) -> Variant (B::String | r)
f = on _A (inj _B) identity
```

## Records

Homogenous records Challenge! - Write a function signature where the input is some arbitrary record and the output is a record with the same fields but all types converted to Number -

Typescript version uses [Index types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#index-types)
```typescript
const f = <O>(_: {[K in keyof O]: number}) => ...
```

[Try it online!](https://www.typescriptlang.org/play?#code/MYewdgzgLgBAhgLhgbzHAtgUydATgSzAHMAaeI7GMAV3QCNNcBfGAXhTSyQHIAzEENzJwKSAEwAGJgG4AULNCRYqDJQD6ZAHTbcmaC3ZwF4aDF5sYAHgDyMTAA8omMABMIMamADWYEAHcwAD4ACjUkZABtAGkYQhgvTABPEHNrAF0kGnpGJgBKNkCUJlleYN1oXKA)

Challenge: Represent a record that does *NOT* have a particular label -

Typescript version uses `never`. The following function will not accept a record with field `foo`.
```typescript
const f = (_: Record<string, number> & {foo?: never}) => {}
```

[Try it online!](https://www.typescriptlang.org/play?#code/MYewdgzgLgBAZjAvDAFAfQFwwEoFNQBOAJgDzQECWYA5gDQxgCuAtgEa4EB8MAZDAN5wQIAPxYwuAG4cAvgEok3fjIBQKuCmVz1mgIZYAjPJ2Dhh+UA)

Challenge: Represent a record which is the union of two arbitrary records -

Typescript again uses index types to great effect! The following function will accept three records where the third must necessarily be a union of the first two -
```typescript
const f = <A extends unknown>(a: A, b: {[K in keyof A]: A[K] | null}) => {}
```

[Try it online!](https://www.typescriptlang.org/play?#code/MYewdgzgLgBAZjAvDAPAQRgUwB5U2AEwhgFcwBrMEAdzAD4AKAQwC4Y0AaGAIzYG8A2gGkYASzAxymAJ4gEaALps0whTAA+MMCQA2OgL4BKJHRh99AKAtwGfOCBBsAjF25MATmwDk0d+IDmXvpcdg5s2nquHt5u7kGG1rb2jjAuPNEwPlB+YIHBZsneETpeUZ6ZsfGJoSlpsd6+AUEhhVq6Bgk2Nc5lDdlN+Xz1FR7xQA)


## Dynamic record manipulation

Challenge: Can we add fields dynamically, and have them be tracked? For example, can you write a function `addField :: Record -> String -> a -> Record` so that the output record is the input record with a field added?

Typescript can do it but by using records instead of fieldnames.

So this doesn't work -
```typescript
const add = <O, K extends string, V>(o: O, k: K, v: V): O & {[K]: V} => ({...o, [k]: v})
const set = <O, K extends keyof O>(o: O, k: K, v: O[K]): O => ({...o, [k]: v})
const res = set(add({a:1, c:'as'}, 'b', 3), 'b', 33)
console.log(res)
```

But this does -
```typescript
const add = <A, B>(a: A, b: B) => ({...a, ...b})
const set = <O, K extends keyof O>(o: O, k: K, v: O[K]): O => ({...o, [k]: v})
const res = set(add({a:1, c:'as'}, {b: 3}), 'b', 33)
const res2 = set(add({a:1, c:'as'}, {b: 3}), 'c', 33) // fails
const res3 = set(add({a:1, c:'as'}, {b: 3}), 'c', 'string works')
const res4 = set(add({a:1, c:'as'}, {b: 3}), 'd', 'string works') // fails
```

## Dynamic Record fieldname manipulation

One of the newer shiny things in typescript is `Uppercase`, `Capitalise` etc.

E.g. -
```typescript
type A = Uppercase<'foo'> // Is the same as type A = 'FOO'
```

Also you can do -
```typescript
type Getter<A extends string> = `get${Capitalize<A>} `
type B = Getter<'fooBar'> // Becomes 'getFooBar'

// And use `never` to disallow
type FromGetter<A extends string> = A extends get${infer B} ? Uncapitalize<B> : never 
type C = FromGetter<'willFail'> 
type D = FromGetter<'getThisWillWork'>;
```

Dynamic field manipulation is possible in Purescript as well but clunkier. See for example https://qiita.com/kimagure/items/4f5c6054870f631ff768.
