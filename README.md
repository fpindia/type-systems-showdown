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

