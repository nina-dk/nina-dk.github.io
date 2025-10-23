---
title: Let's X-plore JavaScript's Object Creation Patterns!
date: 2021-07-16 20:34:23
categories:
- JavaScript
- Object
tags:
- OOP
- Object
- Prototypal Inheritance
---

In an attempt to fully absorb the various object creation patterns in JavaScript, in this article, we’re going to explore them through an example.

{% img https://images.unsplash.com/photo-1612036782617-472725567302?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=1170&q=80 %}

<!-- more -->
Let’s define them, first, though:

**1. Object factories or factory functions pattern**
**2. Constructor/prototype or pseudo-classical object construction pattern**
**3. Objects Linking to Other Objects (OLOO) pattern**
**4. ES6 Classes**

Okay. So we’ll create some of the X-Men using all of the different patterns to see, in practice, how they work, as well as what their differences and similarities are.

## 1. Object factories

Object factories (aka factory functions) are perhaps, the simplest object creation pattern. We define a function that returns an object that has the properties and methods that we want our instances to have. If we want to provide some properties with custom values (i.e. have our instances have a unique state), we can achieve that by passing the values as arguments to the function.

{% gist 2bbe7820f1b850f3793ee234ef931e75 factoryFuncXMen.js %}

In this example, we create a `createXMen` factory function that returns an object with two properties and two methods. In the `createWolverine` and `createJean` factories, we use the `Object.assign` method to endow the instances of the factories with the properties and methods that `createXMen` provides, as well as assign some unique state and functionality to them.

This way, we can invoke `callProfessor` and `displayPowers` on both `wolverine` and `jean`. One drawback of this paradigm is that the objects created by all the above factory functions have their own copies of all of the methods (as displayed on line 45).

``` javascript
Object.getOwnPropertyNames(jean); // ['name', 'powers', 'callProfessor', 'displayPowers', 'enterPhoenixMode']
```
 Now this is just a small example, but imagine having thousands of X-Men objects; it would put a pretty heavy load on system memory.

Another drawback would be that we can’t determine what types of objects `wolverine` and `jean` are (line 46).

```javascript
jean instanceof createJean; // false
```

The `instanceof` operator doesn’t work with factory functions. Why? Well, according to the [MDN docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof): “The `instanceof` operator tests to see if the prototype property of a constructor appears anywhere in the prototype chain of an object.” Since there’s no inheritance relationship between an object factory and its instances, instanceof is of no use to us here.

## 2. Constructor/prototype pattern
Also known as the *pseudo-classical object construction pattern*, this pattern utilizes constructor functions and their prototype property to create objects of the same type that inherit their behaviour (methods) from the constructor function’s prototype. We can also have constructors inherit from other constructors by reassigning the “child” constructor’s prototype property to reference an object that inherits from the “parent” constructor’s function prototype using `Object.create`, as seen below (lines 19 and 29):

{% gist eebfbcf4a7650d049a3913eba28a0abe constructorsXMen.js %}

A lot is going on here, but we’re just going to highlight the most important parts:

1. We invoke constructors using the `new` operator. `new` implicitly creates and returns a new object, without us having to return anything from the constructor explicitly. The properties that we want to add to this new object are added inside the constructor because `new` also sets the binding for `this` in the constructor to the newly created object.
In addition to that, `new` sets the internal `[[Prototype]]` property of the instances of the constructor to point to the object referenced by the constructor’s prototype property. In plain words, the instances inherit from the constructor function’s prototype. Therefore, we can add properties and methods in `Jean.prototype`, for example, and all instances of `Jean` will have access to them. This is why we can invoke `enterPhoenixMode` on `jean`, even though the object itself doesn’t define such a method, as seen on line 48.

```javascript
console.log(Object.getOwnPropertyNames(jean)); // ['name', 'powers']
```

2. As mentioned before, we can have subtypes: a constructor that inherits from a parent constructor. In this example, `Wolverine.prototype` and `Jean.prototype` inherit from `XMen.prototype`. Thus, instances of `Wolverine` and `Jean` can delegate method calls to their granddaddy, `XMen.prototype` (lines 38 and 45).

```javascript
//`displayPowers` is defined on XMen.prototype...
wolverine.displayPowers();
//Logan has claws
//Logan has superhuman senses
//Logan has healing factor

//omitted code

//...as is `callProfessor`
jean.callProfessor(); // Calling Charles...
```

Notice lines 20 and 30? Before reassigning `Wolverine.prototype` and `Jean.prototype` to these new objects that inherit from `XMen.prototype`, they both pointed to another object that owned a `constructor` property, which pointed back to the constructor function, `Wolverine` and `Jean`, respectively.

{% asset_img constructorProp.png "Constructor property'The constructor property before and after reassigning the function prototype to a new object'" %}

When we reassign `Jean.prototype` to reference a new object that inherits from `XMen.prototype` the old object along with its belongings is garbage collected. As a consequence, when we reference `constructor` on `Jean.prototype` on line 33 in the picture above, it no longer references the expected function object. In reality, there is no longer a constructor property in the object referenced by `Jean.prototype`. Therefore, JS looks for it in the object’s prototype, `XMen.prototype`, which (like any constructor function’s prototype) defines that property. However, `XMen.prototype.constructor` points to `XMen`, so that’s what `Jean.prototype.constructor` also evaluates to (line 34). To fix that, we manually add a `constructor` property in the new `Jean.prototype` and have it point to `Jean`. This way if we later want to find out who created an instance of `Jean` we can simply access `jean.constructor` (where `jean` is an instance of `Jean`):

{% asset_img jeanConstructor.png "The constructor of the jean object is Jean'Accessing constructor on jean gives us the constructor function that created it
'" %}

## 3. OLOO

The *Objects Linking to Other Objects* pattern works in a somewhat different way than the rest of the patterns we’ve seen: it doesn’t use functions; instead, it creates a prototype object from which all objects of a given type will inherit their state and methods. We can differentiate the several instances' state by using a method usually called `init`, and passing the values we want as arguments to it. Better explained with an example:

{% gist 6fed14f4b8a6b63aee91149f9a55be2f OLOOXMen.js %}

In this paradigm, `XMen`, `Wolverine` and `Jean` are merely objects themselves. To have their instances inherit from them we leverage the `Object.create` static method, which creates an empty object and sets its internal `[[Prototype]]` property to point to the object it receives as an argument. On line 45, we instantiate a new object referenced by `jean` that inherits properties and methods defined on `Jean`.

```javascript
let jean = Object.create(Jean).init();
```

Okay, and what about `.init()`? The `init` method (called this by convention) is necessary to also add *state* (properties) to our instances. Let’s break down lines 2–6:
```javascript
init(name, powers) {
  this.name = name;
  this.powers = powers;
  return this;
}
```
There’s this `init` method defined on `XMen`, which takes two arguments, `name` and `powers`. In the method, we define two properties also called `name` and `powers` on the object referenced by `this` when we invoke `init`, and assign them the arguments passed in as values. This method is the “simulation” of the constructor function in the pseudo-classical construction pattern. We invoke `init` on an instance of `XMen` and endow it with a personalized state.

Now, if we need to create a sub-type of `XMen` like we did before, we can create an object that inherits all the methods from `XMen`, again using `Object.create` (lines 17 and 31). Subsequently, we add Wolverine-specific methods directly on the object referenced by `Wolverine` (lines 19–29). Note that I used `Object.assign` simply to avoid defining all the methods on `Wolverine` one by one, but that’s not necessary.

{% asset_img subtypeOLOO.png "The init method of the Wolverine prototype object'The init method of the Wolverine object'" %}

There’s quite a lot going on on line 21, so let’s clear this up, too:

Remember how we invoke the parent constructor on line 15 of the constructor/prototype pattern to add the state it provides to the instances of `Wolverine`?

```javascript
XMen.call(this, "Logan", ["claws", "superhuman senses", "healing factor"]);
```

That’s what we’re doing on line 21 of this example, as well. However, now `XMen` is just an object, so what we need to invoke is the `init` method of the `XMen` object, which is the one responsible for adding state to the instances of `XMen`.

Finally, on line 58, we can see that `instanceof` cannot be used to determine the type of an instance in this paradigm either. However, a workaround is using `Object.getPrototypeOf` to at least determine whether an object is the prototype of an instance.

{% asset_img OLOOlink.png "Use Object.getPrototypeOf to determine the type of an object'Access jean’s prototype to determine its type, indirectly'" %}

## 4. ES6 Classes

Classes are little more than syntactic sugar; to their core, they’re just a variation of the pseudo-classical creation pattern. Classes in JavaScript are not even a distinct data type, they’re just function objects. However, they are easier to use as they abstract some of the implementation details of the constructor/prototype pattern away. For example, all we have to do to have a class’s prototype inherit from a superclass’s prototype is use the `extends` keyword after the class’s name. So much for reassigning the function prototype to an object that inherits from the supertype's prototype property. Back to our example:

{% gist 250220644d163765e7e467c610c007a7 classXMen.js %}

Definitely a lot cleaner. So, a class’s `constructor` method works like a constructor function (hence the name, I suppose): it’s the one responsible for adding properties to the class instances. The methods we define below it, inside the class body, are instance methods and are defined on the object referenced by the class’s prototype property behind the scenes. For example, `attack` lives in `Wolverine.prototype`.

The `extends` keyword does what lines 19 and 29 do in the constructor/prototype example. Redefining the constructor property in the subclass’s prototype is no longer necessary — JS deals with that for us.

{% asset_img wolverineClass.png "Subclass inherits from superclass using the extends keyword in JavaScript'Wolverine’s prototype inherits from XMen’s prototype property'" %}

The `super` keyword, on line 18, functionally replaces line 15 in the constructor/prototype pattern example. When we invoke `super` in the `constructor` method of the child class, it invokes the `constructor` method of the parent class (`XMen`) and adds the properties defined in it to the object currently referenced by `this`: the new instance of `Wolverine`.

The methods we define after the `constructor` special method are defined on the function prototype of each class, in the same way as if we’d done: `Wolverine.prototype.attack = function() {...}`. This means that all instances of `Wolverine` and `Jean` delegate their method calls to their prototype, `Wolverine.prototype` and `Jean.prototype`, respectively, instead of carrying around their own copies of each method.

### Compare and contrast

- Constructors and classes both create objects that trace back to their creators easily using the `instanceof` operator or by accessing the `constructor` property on the instances. They’re also memory efficient since their instances inherit (usually all of) the methods from their `prototype` property.
- Object factories help us create multiple objects of the same type, without being able to determine whether they are of that given type, afterwards. They're also memory intensive as each of their instances holds its own copies of all the methods.
- The OLOO pattern lies somewhere in between: it’s more effective than object factories since the prototype object that we create is the one that defines all the necessary methods that the instances will then access through prototypal delegation. However, it doesn’t provide a very straightforward way to identify the type of the instances of the prototype object, like classes and constructors do.