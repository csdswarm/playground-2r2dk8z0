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
  try {
    console.log(`I am a prototype method created that is attached and has access to the current version of this object, however, I only have access to properties attached to this so this.first "${this.first}"`);
    console.log(`but not last "${last}", which will error.`)
  } catch (e) {
    console.log(`last triggered an error, but ... proto functions are good memory savers as they are referenced rather than copied.`);
  }
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

One main issue to be aware of in JS prototypes is that they are malleable. This makes them quite useful in a number of scenarios that are much
more difficult to create in typical OOP environments that have real classes, but it has some drawbacks, too. 

For one, it can make it easier to modify (hack) things that weren't intended to be modified. 

For instance. Let's say you've defined a prototype in your library and make it available for other code and libraries to use. What you may not know
is that, without some additional effort, these things can be pretty easy to take control of. 

Watch this:

```javascript runnable
function MyObj(first, last) {
  this.first = first;
  this.last = last;
}

MyObj.prototype.sayMyName = function () {
  console.log(`I'm ${this.first} ${this.last}`);
}

// In one library you might have this like this:
const myObj = new MyObj('Original', 'Gangsta');
myObj.sayMyName();

// In a library that has been given access, however, we have this:
const hackerObj = new MyObj('MC', 'Hammer');
hackerObj.sayMyName();

// This guy can actually override the prototype above, however
hackerObj.__proto__.sayMyName = () => console.log(`It's Hammer time`);

myObj.sayMyName();
hackerObj.sayMyName();
```

That could create some havoc, right?

### Prototypal inheritance

So, this is the main point of creating all these constructors. It's basically so that we can use inheritance to reduce our work for similar
things that are similar.

Why write the same methods on multiple different classes that all have some base characteristics in common.

```javascript runnable
function MyObj(first, last) {
  this.first = first;
  this.last = last;
}

function sayMyName() {
  console.log(`Hi, I'm ${this.first} ${this.last}`);
}

Object.assign(MyObj.prototype, { sayMyName });

function MyImprovedObj(mi, ...rest) {
  MyObj.apply(this, rest);
  this.mi = (mi||'')[0];
}

function sayMyFullname() {
  console.log(`Hi, my full name is ${this.first} ${this.mi}. ${this.last}!!!`);
}

// we extend by setting the prototype of our object to an instance of the prototype of the base class.
// since we're here, lets hookup the other prototype methods, while we're at it.
MyImprovedObj.prototype = Object.assign(Object.create(MyObj.prototype), {
  sayMyFullname
});

const batman = new MyImprovedObj('Thomas', 'Bruce', 'Wayne');
batman.sayMyName();
batman.sayMyFullname();

// we can also see that this is an instance of either constructor, which can also be useful
console.log(batman instanceof MyImprovedObj);
console.log(batman instanceof MyObj);
```

I do want to show one interesting thing about prototypes that may make them a bit easier to understand, at least with what's going on. 

This will help you understand the difference between the prototype and the instance.


```javascript runnable
function Super() {}

Object.assign(Super.prototype, { a: 0, x: 1, y: 2, showMe() { console.log(JSON.stringify(this, 2, null)) } });

function Child() {}

// we extend by setting the prototype of our object to an instance of the prototype of the base class.
// since we're here, lets hookup the other prototype methods, while we're at it.
Child.prototype = Object.assign(Object.create(Super.prototype), {
  a: 11, z: 3
});

const test = new Child();
test.showMe();
```

Now let's talk about 

## Classes

Classes provide a slightly cleaner syntax for our data than prototypes. 

If we are talking about "true" classes and what is commonly referred to as Object Oriented Programming (OOP), then there are a few principles
that basically govern this paradigm:

Encapsulation:
This is basically all about keeping your private data private and your public data public. True OOP classes have this. Private members cannot be
accessed and modified outside of the class declaration itself.

Abstraction:
In short, this is about hiding away complexity, and it is often expressed in inheritance patterns, but can also be expressed by simply taking our
larger pieces of code and breaking them down into smaller, more manageable functions.

Inheritance:
We've already touched on this, but the general idea is to provide a means to share common functionality through the extension of base classes.
A Motorcycle, Car and SUV are not the same thing, but they do tend to have a lot of similar features such as engines, wheels and ignitions and
they are all primarily used for transportation. All things that could be summed up as "vehicles".

Polymorphism:
Basically the ability to take on many forms. This concepts provides classes to have methods that are defined not only by name, but also by their
signature and who they belong to. This is often accomplish through concepts such as overriding and overloading of methods and properties.

JS has all of these things, however, they aren't exactly baked into the language like they are in many classical OOP languages, and often by
applying JavaScript's facsimile of classes in other languages, we actually make it more difficult to have some of these things 
(primarily encapsulation as shown above)

Let's get started with some examples of classes in JS:


```javascript runnable
class MyObj {
  constructor(first, last) {
    this.first = first;
    this.last = last;
  }

  sayMyName() {
    console.log(`Hi, I'm ${this.first} ${this.last}`);
  }
}

class MyImprovedObj extends MyObj {
  constructor(mi, ...rest) {
    super(...rest);
    this.mi = mi;
  }

  sayMyFullname() {
    console.log(`Hi, my full name is ${this.first} ${this.mi}. ${this.last}!!!`);  
  }
}

const superman = new MyImprovedObj('Joseph', 'Clark', 'Kent');
superman.sayMyName();
superman.sayMyFullname();

// Definitely cleaner that what we had before, but ...
const spiderman = new MyObj('Peter', 'Parker');
spiderman.sayMyName();
spiderman.__proto__.sayMyName = () => console.log('There is no Dana, there is only Zuul.');

spiderman.sayMyName();
superman.sayMyName();

// Even though we "say" we are using "classes", we are still really using prototypes. 
```

As you can see in the example above, the syntax is much cleaner, but we aren't really using classes. 

But

### What are the differences between classes and prototypes?

Classes are actually supposed to be more like "blueprints" for new objects, whereas prototypes are objects themselves that get copied.

Consider the difference between a printing press and a photocopier. A printing press often requires a bit more effort to set up initially for
a print, but once set up, it can easily make many prints of a particular template. A photocopier, on the other hand takes an original and simply
makes a copy of it. The setup basically involves putting the page to copy on top of the scanner and voila.

Similar to printing presses, classes require a bit more effort to change the output of. Like photocopiers, with prototypes, if someone can access
the original copy, they can white out some parts and add their own stuff in and all future copies will be affected. Of course, it's not an exact
comparison, because changing the original won't actually change existing copies with a photocopier, but they will with a prototype.





