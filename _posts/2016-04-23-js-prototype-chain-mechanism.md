---
layout: post
permalink: /js-prototype-chain-mechanism
title: JS Prototype chain mechanism
path: 2016-04-23-js-prototype-chain-mechanism.md
time: 5
---

This is my second post in the series of "JS: Leave the classes to those other languages". If you haven’t checked out the first post, you can read it on [JS: Leave the classes to those other languages](/js-leave-the-classes-to-those-other-languages).

### Objects

In JavaScript _almost_ everything is an object. But what are objects in JS?

* Objects are collections of _key/value_ pairs
* Objects can be declared essentially in two ways: a _literal_ or a _constructed_ form
* Every object is linked internally to another object (its _prototype_)

### The literal form

The literal syntax for an object looks like this:

{% highlight javascript %}
var car = {
  color: 'red';
};
{% endhighlight %}

And here is a mental map of what actually happens:

![Literal form](http://i.imgur.com/sL9KEme.jpg)

JS is, of course, creating an object `car` with a color property 'red' but it is also automatically creating an internal link between `car` and `Object.prototype`. `Object.prototype` contains all the built-in methods (i.e. `toString()`) which every object normally can use and it becomes in our case the `car` object prototype. JS do automatically this for us when we use the literal form.

### The constructed form

The constructed form to obtain the same sort of object looks like this:

{% highlight javascript %}
var car = new Object();
car.color = 'red';
{% endhighlight %}

And here is a mental map of what actually happens:

![Constructed form](http://i.imgur.com/2rP4eLG.jpg)

Here things start to be no more immediate. The first time I saw this code I was almost sure to create an instance of the `Object` class and assign it to the variable `car`. I was far away from what happens.

__JS does not have classes__. `Object` is just a function and I'm using this function as a constructor here.
It sounds contorted and it actually is. Personally I don’t like to use the `new` keyword because it was written into the language to act more like Java and I find its use very confusing. __We are not working with classes here__.

Let's break down this code to have a better understanding of what's going on behind the scenes.

#### What is a function?

A function is a sub-type of object (technically, a "callable object"). Functions in JS are said to be "first class" in that they are basically just normal objects, and so they can be handled like any other plain object.

{% highlight javascript %}
function Vehicle() {
  this.color = 'red';
}
{% endhighlight %}

![Function](http://i.imgur.com/9X1MLsL.jpg)

Like any object a function can have properties. Functions automatically get a property called `prototype`, which is just an empty object.

#### What is a constructor?

A constructor is __any function which is used as a constructor__.

JS doesn’t make a distinction. A function can be written to be used as a constructor or to be called as a normal function, or to be used either way. I'm using here the `Vehicle` function as a constructor putting the `new` operator before the function call.

{% highlight javascript %}
var myVehicle = new Vehicle();
{% endhighlight %}

What happens when the constructor is called?

![Constructor call](http://i.imgur.com/OT5vvcD.jpg)

1. It creates a new object `myVehicle`
2. It sets the constructor property of the object to `Vehicle`
3. It sets up the object to __delegate__ to `Vehicle.prototype`
4. It calls `Vehicle()` in the context of the new object

### The ES5 Object.create() method

EcmaScript 5 introduces a much simpler and more straightforward way to create an object.
The `Object.create()` method creates a new object with the specified prototype.
We can specify a particular prototype for a new object without using any function as a constructor.

{% highlight javascript %}
var myVehicle = Object.create(Vehicle.prototype);
myVehicle.color = ‘red’;
{% endhighlight %}

If you need to support pre-ES5 environments (like older IE's), here is a simple partial __polyfill__ for `Object.create()` that gives us the capability that we need even in those older JS environments.

{% highlight javascript %}
if (!Object.create) {
    Object.create = function(o) {
        function F(){}
        F.prototype = o;
        return new F();
    };
}
{% endhighlight %}

This polyfill works by using a _throw-away_ `F` function and overriding its `.prototype` property to point to the object we want to link to. Then we use `new F()` construction to make a new object that will be linked as we specified.

`Object.create()` method will be the main character in [OLOO style pattern](/alternative-patterns-in-JS-OLOO-style).

### The prototype chain

Once understood how to declare objects we need to learn how JS retrieves properties from objects. It uses the prototype chain mechanism I explain here through a practical example. Here is the code:
{% highlight javascript %}
var shape = {
  border: ‘red’
};

var triangle = Object.create(shape);

triangle.border; // 'red'
triangle.area; // undefined
{% endhighlight %}

Here is a mental map of the prototype chain generated by the code above.

![Prototype chain](http://i.imgur.com/X3OFDDf.jpg)

When we try to retrieve the `border` property of `triangle`, JS firstly looks up in the `triangle` object. If it can't find such a property, it tries to retrieve the property looking in the `triangle`'s prototype. In our case `triangle`'s prototype is the `shape` object and JS just returns the value contained in the `shape` `border` property ('red' in our case).

Similar things happen when we try to retrieve a property the prototype chain does not contain. JS looks through all the chain until it finds the primitive `null`; at this point it returns `undefined` because it has not been able to find any property with such a key name in the whole prototype chain.  

<br>

Now that we got a general understanding of how we can declare an object and how the prototype chain works we can carry on and explore [pseudoclasses in JS](/pseudoclasses-and-prototypal-inheritance-in-JS).
