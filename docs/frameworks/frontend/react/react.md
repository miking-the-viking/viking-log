# [React](https://reactjs.org/)

Let's check out React, bitches!
## Intro

### Caveats and Prerequisites of Note


>**A Note on `jsx` and `tsx`**
>
>`jsx` provides an embeddable XML-list syntax into JavaScript, by extrapolation `tsx` is TypeScript that supports the XML-like syntax.
>
>In the `.tsconfig` the `jsx` option must be enabled.
>
>```json
>"jsx": "preserve", /* Specify JSX code generation: 'preserve', 'react-native', or 'react'. */
>```
>
>- `preserve` will keep the JSX as part of the output to be consumed by another transformation step (like Babel), and the output will have a `.jsx` extension
>- `react` will emit a `React.createElement`, doesnot need to go through a JSX transformation before use, and the output will have a `.js` extension.
>- `react-native` mode is equivalent of `preserve` in that it keeps the `JSX` but the output will have a `.js` file extension
>
>
---
>**A Note on the **`as`** operator**
>
>Type assertion is writte thusly:
>
>```typescript
>var foo = <foo>bar;</foo>
>```
>
>However JSX uses the angled brackets, in order to avoid these parsing diffficulties in `tsx`: angle bracket type assertions are disallowed.
>
>Instead make use of the new `as` operator:
>
>```typescript
>var foo = bar as foo;
>```
---

### Type Checking

>**Intrinsic vs value-based elements**
>
>In React, intrinsic elements are emitted as strings using `React.createElement("div")`, while components authored by the programmer are treated differently: `React.createElement(MyComponent)`
>
>The types of attributes passed in the JSX elements are looked up differently than intrinsic element attributes. Intrinsic attributes should be known _intrinsically_ whereas custom components will specify their own set of attributes.
>
>TypeScript uses the same convention as React does to distinguish between the two: intrinsic elements always begin with a lowercase letter, and value-based elements always begin with an uppercase letter.

#### Intrinsic Elements

Intrinsic elements are looked up on a special interface: `JSX.IntrinsicElements`. By default, if this interface is not specified then anything is permitted and intrinsic elements will not be type checked. **If this interface _is_ present, then the name of the intrinsic element is looked up as a preperty on the `JSX.IntrinsicElements` interface.**

```typescript
declare namespace JSX {
    interface IntrinsicElements {
        foo: any
    }
}

<foo />; // ok
<bar />; // error, not specified in the `JSX.IntrinsicElements` interface
```

>**Catch-all string indexer**
>
>```typescript
>declare namespace JSX {
>   interface IntrinsicElements {
>       [elemName: string]: any;
>   }
>}
>```

#### Value-based elements

Value-based elements are looked up by identifiers that are in scope.

```typescript
import MyComponent from './myComponent';

<MyComponent />; // ok
<SomeOtherComponent />; // error
```

Value-based elements can be defined in two ways:

1. Stateless Functional Component (SFC)
2. Class Component

##### Stateless Functional Component

An SFC is defined as a JavaScript function where its first argument is a `props` object, with an enforced return type that must be assignable to `JSX.Element`.

```typescript
interface FooProp {
    name: string;
    X: number;
    Y: number;
}

declare function AnotherComponent(prop: {name: string});
function ComponentFoo(prop: FooProp) {
    return <AnotherComponent name=prop.name />;
}

const Button = (prop: {value: string}, context: { color:string }) => <button>
```

Since SFC is simply a JacaSvript function, it overloading can be applied as well:

```typescript
interface ClickableProps {
    children: JSX.Element[] | JSX.Element
}

interface HomeProps extends ClickableProps {
    home: JSX.Element
}

interface SideProps extends ClickableProps {
    side: JSXElement | string;
}

function MainButton(prop: HomeProps): JSX.Element;
function MainButton(prop: SideProps): JSX.Element {
    ...
}
```

##### Class Component

It's possible to limit the type of a class component, introducing two more terms: _element class type_ and _element instance type_.

Given `<Expr />`, the _element class type_ is the type of `Expr`, however if `Expr` was a factory function, the class type would be that function.

Once the _class type_ is established, the _instance type_ is determind by the union of the return types of the class tyoe's call signatures and construct signatures. For an ES6 class, the _instance type_ would be the type of an instance of that class, and for a factory function it would be the type of the value returned from the function.

```typescript
class MyComponent {
    render() {}
}

// use a construct signature
var myComponent = new MyComponent();

// element class type => MyComponent
// element instance type => { render: () => void }

function MyFactoryFunction() {
    return {
        render () => {}
    }
}

// use a call signature
var MyComponent = MyFactoryFunction();

// element class type => FactoryFunction
// element instance type => { render: () => void }
```

The _element instance type_ is interesting because it must be assignable to `JSX.ElementClass` or an error will be thrown. By default `JSX.ElementClass` is `{}`, but it can be augmented to limit the use of JSX to only those tpyes that conform to the proper interface.

```typescript
declare namespace JSX {
    interface ElementClass {
        render: any;
    }
}

class MyComponent {
    render() {}
}

function MyFactoryFunction() {
    reeturn { render: () => {} }
}

<MyComponent />; // ok
<MyFactoryFunction />; // ok

class NotAValidComponent {}
function NotAvalidFactoryFunction() {
    return {};
}

<NotAValidComponent />; // error
<NotAValidFactoryFunction />; // error
```

#### Attribute Type Checking

To type check attributes, the _element attributes type_ must be determined. This is different between _intrinsic_ and _value-based_ elements.

_Intrinsic element_ types are the type of a property on `JSX.IntrinsicElements`

```typescript
declare namespace JSX {
    interface IntrinsicElements {
        foo: { bar?: boolean }
    }
}

//elements attributes type for `foo` is `{ bar?: boolean }`
<foo bar />;
```

_Value-based_ elements are a bit more complex. Type is determined by the type of a peroperty on the _element instance type_ that was previously determined. The property to use is determined by `JSX.ElementAttributesProperty` which should be declared with a single property. The name of that property is then used.

```typescript
declare namespace JSX {
    interface ElementAttributesProperty {
        props; // specify the property name to use
    }
}

class MyComponent {
    // specify the property on the lement instace type
    props: {
        foo?: string;
    }
}

// element attributes type for `MyComponent` is `{ foo?: string }`
<MyComponent foo="bar" />
```

The _element attribute type_ is used to type check the attributes in the JSX. Both **optional** and **required** properties are supported:

```typescript
declare namespace JSX {
    interface IntrinsicElements {
        foo: { requiredProp: string; optionalProp?: number }
    }
}

<foo requiredProp="bar" />; // ok
<foo requiredProp="bar" optionalProp={0} />; // ok
<foo />; // error, requiredProp is missing
<foo requiredProp={0} />; // error, requiredProp should be a string
<foo requiredProp="bar" unknownProp />; // error, unknownProp does not exist
<foo requiredProp="bar" some-unknown-prop />; // ok, because 'some-unknown-prop' is not a valid identifier

// using the spread operator

var props = { requiredProps: "bar" };
<foo {...props} />; // ok

var badProps = {};
<foo {...badProps} />; // error

```

#### Children Type Checking

`children` is a property in an _element attributes type_ which has been determined by type checking attributes. Similar to using `JSX.ElementAttributesProperty` to determine the name of _props_, `JSX.ElementChildrenAttribute` is used to determine the name of `children`.

`JSX.ElementChildrenAttributes` should be declared with a single property.

```typescript
declare namespace JSX {
    interface ElementChildrenAttribute {
        children: {}; // specify children name to use
    }
}
```

Without explicitly defining the type of children, default types will be used from React typings.

The type of `children` can be specified liek any other attribute and will override the default type from REact typings.

```typescript
interface PropsType {
    children: JSX.Element
    name: string
}

class Component extends React.Component<PropsType, {}> {
    render() {
        return (
            <h2>
                this.props.children
            </h2>
        )
    }
}

// OK
<Component>
    <h1>Hello World</h1>
</Component>

//Error: children is of type JSX.Element, not an array of JSX.Element
<Component>
    <h1>Hello World!</h1>
    <h1>Hello World!</h1>
</Component>

//Error: children is of type JSX.Element, not array of JSX.Element or string
<Component>
    <h1> Hello</h1>
    World
</Component>
```

### The JSX Result Type

By default, the result of a JSX expression is typed as `any`. This can be customized by specifying the `JSX.Element` interface. **However, it is not possible to retrieve type information about the element, attributes or children of the JSX from this interface. It is a _black box_.**

### React Integration

To use JSX with React, use [React Typings](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts), these typings define the `JSX` namespace appropriately for use with React.

