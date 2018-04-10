# [TypeScript](https://www.typescriptlang.org/docs/home.html)

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

