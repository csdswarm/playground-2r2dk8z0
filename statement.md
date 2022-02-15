# Prototypes, Classes and Object Literals in JS!

Javascript is pretty flexible in the types of object like structures you can use. However, it can also get very confusing.

In this session, we are going to go over a few different ways of working with object-like structures, will explain a few of the differences
and help clarify what's different in JS.

## Prototypes

Let's start with prototypes. These are the base class/object like structure used inside of JS. While technically all of our object like structures
are going to be prototypes in JS, we will compare the different usages of them.

Here is perhaps the most core instantiation of a prototype in JS

```javascript runnable
const myObj = Object.create(Object);

console.log(myObj)
```

You may have noticed that what was logged was `Function {}`

That's because Function is the main constructor function from which Object is created. In JS, functions are multi-purpose. They can be used for
subroutines, functions, methods and object creation. For now, we will focus on how a function can be used as a constructor for an object instance.

```javascript runnable
function MyObj(first, last) {
    this.first = first;
    this.last = last;
}

const myObj = new MyObj('joe', 'schmoe');

console.log(myObj)
```
