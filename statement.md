# Prototypes, Classes and Object Literals in JS!

Javascript is pretty flexible in the types of object like structures you can use. However, it can also get very confusing.

In this session, we are going to go over a few different ways of working with object-like structures, will explain a few of the differences
and help clarify what's different in JS.

## Prototypes

Let's start with prototypes. These are the base class/object like structure used inside of JS. While technically all of our object like structures
are going to be prototypes in JS, we will compare the different usages of them.

I'll go into more detail on the differences between prototypes and classes later, but for now, I'll say that prototypes work basically by
making a "copy" (more or less), of an existing object, rather than creating a new object based on rules. For the most part, we are splitting
hairs to make this distinction, however, there are times when it becomes important to understand later on.

Here is perhaps the most core instantiation of a prototype in JS

```javascript runnable
const myObj = Object.create(Object);

console.log(myObj)
```

You may have noticed that what was logged was `Function {}`

That's because `Function` is the main constructor function from which Object is created. In JS, functions are multi-purpose. They can be used for
subroutines, functions, methods and object creation. For now, we will focus on how a function can be used as a constructor for an object instance.

```javascript runnable
function MyObj(first, last) {
    this.first = first;
    this.last = last;
}

const myObj = new MyObj('joe', 'schmoe');

console.log(myObj)
```

Now, you will see that the constructor was `MyObj` instead of `Function`. What differentiated this from a typical function is basically only
the `new` keyword used when it was called. Otherwise, it's really not much different than any other function in JS.

Even the `this` that is utilized within the function doesn't automatically make it a `class` type structure. Watch this:

```javascript runnable
function MyObj(first, last) {
    this.first = first;
    this.last = last;
}

function OtherObj() {}
OtherObj.prototype.myObj = MyObj;

const myObj = new OtherObj();

console.log(myObj);
myObj.myObj('bozo', 'leClown');

console.log(myObj);
```

As you can see, the `MyObj` function is quite flexible and can be used both for instantiating a new object as well as being a method on another
object prototype.

Of course, you can make things really fun (and confusing), by actually checking how the function was called

```javascript runnable
function MyObj(first, last) {
    if(new.target) {
        this.first = first;
        this.last = last;
    } else {
        return new MyObj(first, last);
    }
}

const myObj = new MyObj('Joe', 'Schmoe');
const byItself = MyObj('Bozo', 'LeClown');

console.log(myObj);
console.log(byItself);
```

As you can see, we can call this function without the `new` operator and get the same result (mostly);

One thing you may have noticed is that we are Pascal casing these methods. It is a common convention in JS to use pascal case for functions that 
are intended to be used like class constructors, and camel case for functions that are intended to be used as methods or subroutines.

Let's start extending our prototype a bit. There are a few different ways of doing this, I will show two and explain the differences.

```javascript runnable
function MyObj(first, last) {
  this.first = first;
  this.instanceMethod = () => {
    console.log(`I am an instance method created for every constructed version of this object. I have access to this.first "${this.first}" and also last "${last}"`)
  }
}

MyObj.prototype.protoMethod = function () {
  console.log(`I am a prototype method created that is attached and has access to the current version of this object, however, I only have access to properties attached to this so this.first "${this.first}", but not last "${last}". But, I'm a good memory saver.`);
}

MyObj.prototype.fatArrowProto = () => console.log(`Fat arrow functions will likely not work as expected here. this.first is "${this.first}", that's because "this" only refers to the scope in which the fat arrow function exists.`)

const myObj = new MyObj('Joe', 'Schmoe');

myObj.instanceMethod();
console.log();
myObj.protoMethod();
console.log();
myObj.fatArrowProto();
console.log();
this.first = '[The fat arrow function has access to this level]';
myObj.fatArrowProto();

```