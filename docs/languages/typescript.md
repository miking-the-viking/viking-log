# [TypeScript](https://www.typescriptlang.org/docs/home.html)

- [Good Book](https://basarat.gitbooks.io/typescript/)

## What it is?
TypeScript is JavaScript with types allowing for static type checking, easier refactoring, etc. It uses the `.ts` extension.

## Compiling TS

Run `tsc [filename].ts` to compile `[filename].js`.

## Benefits

### Type Annotations

Type annotations are lightweight ways to record the intended contract of the function or variable. 

```typescript
function greeter(person: string) {
    return "Hello, " + person;
}

let user = [0, 1, 2];

document.body.innerHTML = greeter(user);
```

If `user` is not a string, an error will be thrown error TS2345: Argument of type 'number[]' is not assignable to parameter of type 'string'.

Even with errors thrown, the `.js` file will still be made. The TypeScript compiler simply is warning you that code may likely not run as expected.

### Interfaces

In TypeScript, two types are compatible if their internal structure is compatible. This allows implementing an interface just by having the shape the interface requires, without explicit `implements` clause.

```typescript
interface Person {
    firstName: string;
    lastName: string;
}

function greeter(person: Person) {
    return "Hello, " + person.firstName + " " + person.lastName;
}

let user = { firstName: "Jane", lastName: "User" };

document.body.innerHTML = greeter(user);
```

### Classes

TypeScript supports new features in JavaScript, such as support for class-based object-oriented programming.

**NOTE:** The use of `public` on arguments to the constructor is a shorthand enabling automatic creation of properties with that name.

```typescript
class Student {
    fullName: string;
    constructor(public firstName: string, public middleInitial: string, public lastName: string) {
        this.fullName = firstName + " " + middleInitial + " " + lastName;
    }
}

interface Person {
    firstName: string;
    lastName: string;
}

function greeter(person : Person) {
    return "Hello, " + person.firstName + " " + person.lastName;
}

let user = new Student("Jane", "M.", "User");

document.body.innerHTML = greeter(user);
```


## [TypeScript Official Documentation Review] (https://www.typescriptlang.org/docs/handbook/variable-declarations.html)

### Types

#### Boolean

```typescript
let isReady: boolean = false;
```

#### Number
All numbers in TypeScript are floating point values given the type `number`.

```typescript
let decimal: number = 6;
```

#### String

Used for textual data, `string`. Enclose in double quotes (`"`) or single quotes (`'`);

```typescript
let color: stirng = "blue";
color = 'red';
```

##### Templates Strings

Template strings can span multiple lines and have embedded expressions. These strings are surrounded by the backtick/backquote and embedded expressions are of the form 
`${ expr }`.

```typescript
let fullName: string = `Bob Bobbington`;
let age: number = 37;
let sentence: string = `Hello, my name is ${ fullName }`

I'll be ${ age + 1 } years old next month.`;
```

#### Array

Arrays can be declared two ways:

##### `[]`

```typescript
let list: number[] = [1, 2, 3];
```

##### Generic Array: `Array<elemType>`

```typescript
let list: Array<number> = [1, 2, 3];
```

#### Tuple

Express an array where the type of a fixed number of elements is known but need not be the same.

```typescript
let x: [string, number];
x = ["hello", 10];
```

#### Enum

`enum` is a way of giving friendly names to numeric values

```typescript
enum Color {Red, Green, Blue}
let c: Color = Color.Green;
```

#### Any

There are some cases when the type of a variable is unknown when writing the application, such as from dynamic content, user input or a third party library. To opt out of type checking here label with the `any` type.

```typescript
let noteSure: any = 4;
notSure = "a string?";
notSure = false;
notSure = 67;
```

#### Void

`void` is the opposite of `any`, the absense of any type at all. Such as for functions with no return value.

```typescript
function warnUser: void() {
    alert('woah!");
}
```

Declaring a variable type `void` is unuseful as it can only ever be `undefined` or `null`.


#### `null` and `undefined`

Both `undefined` and `null` have their own types, self-named.

By default both are subtypes of all other types, so you can assign `null` and `undefined` to a `number`, for instance.

> **Note:** To avoid _many_ common errors: when using the `--strictNullChecks` flag, `null` and `undefined` are only assignable to `void` and their respective types.
> So in the case where you can pass in either a `string`, `null`, or `undefined`, use the union type: `string | null | undefined`.

#### Never

`never` type represents the type of values that never occur. For instance a function that throws an exception or one that never returns. Variables can also acquire the type `never` when narrowed by any type guards that can never be true.

`never` is a subtype of, and assignable to, every type. To the extreme, no type is a subtype of, or assignable to, `never` except `never` itself; even `any`.

**A function returning `never` must have a reachable end point (ie. no infinite while loops)**

```typescript
function error(message: string): never {
    throw new Error(message);
}

function fail() {
    return error('Something failed");
}
```

### Type assertions

Sometimes you, the programmer, will know more about a value than TypeScript, here is when _type assertions_ come in to play to essentially type cast the variable exclusively for the compiler.

There are two forms of type assertions:

- Angle bracket syntax

```typescript
let someValue: any = "this is a string";
let strLength: number = (<string>someValue).length;
```

- `as` syntax

```typescript
let someValue: any = "this is a string";
let strLength: number = (someValue as string).length;
```

Both forms are equivalent, however when using TypeScript with JSX only `as` style assertions are allowed.

## Variable Declarations

### `let`

#### Scoping

`let` is a newer element introduced to JavaScript that uses _lexical-scoping_ or _block-scoping_, unlike var that uses _function-scoping_.

#### Redeclaration and Shadowing

`let` prevents redeclaration of the same _block_scoped_ variable name by throwing an error, unlike `var` which allows redeclaration of a variable.

#### Block-scoped Variable Capturing

`let` declarations have a drastically different behaviour when declared as part of a loop, or asynchronous function, than a `var`. These declarations create a new scope _per iteration_!

### `const`

A `const` is a variable that is _immutable_ with the same scoping rules as a `let` declaration.

### Decorators

[TypeScript Documentation](http://www.typescriptlang.org/docs/handbook/decorators.html)

The existence of Classes in TypeScript and ES6 leads to new scenarios requiring additional features to support annotating or modifying classes and class members. Decorators provide a way to add both annotations and meta-programming syntax for class declarations and members..

**Note**: Decorators are presently experimental (Sunday April 22 2018), to enable them `enableDecrators` compiler option must be enabled in the `tsconfig.json`.


A _Decoratror_ is a special king of declaration that canbe attached to a class delaration, method, accessor, property, or parameter.

Decorators use `@expression`, where `expression` must evaluate to a function that will be called at runtime with information about the decorated declaration.

```typescript
function someDecorator(target) {
    //do something with target
}

....
@someDecorator....
...
```

#### Decorator Factories

To customize how a decorator is applied to a declaration, use a declarator factory: a function that returns the expression that will be called by the decorator at runtime.

```typescript

function color(value: string) { // the decorator factory
    return function (target) { //this is the decorator
        // do something with target and value
    }
}
```

#### Decorator Composition

Multiple decorators can be declared either on a single line, or multiple lines:

```typescript
@a @b x

@a
@b
x
```

Decorators are evaluated first to last (top to bottom), but the results are returns last to first ( bottom to top).

#### Class Decorators

Class decorators are used just before a class declaration. It gets applied to the constructor of the class and can then be used to observe, modify, or replace a class definition.

**Note:** Class decorators cannot be used in a delcaration file, or in any other ambient context (Such s on a `declare` class).

The expression of the class decorator will be called as a functino at runtime, with the contsructor of the decorated class as its only argument.

If the class decorator returns a value, it will replace the class declaration with the provided constructor function.