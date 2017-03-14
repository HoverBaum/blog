title: Object-oriented JavaScript
tags:
  - js
  - oop
id: 107
categories:
  - English
  - Webdev
date: 2014-07-25 19:54:29
---

### **Attention: This is kinda "old" and I personally now consider this to bad style so I won't make the effort to reformat this after moving from Wordpress to Hexo.**

<!-- more -->

This Tutorial strives to enable you to build object oriented applications in JavaScript. Another goal of mine is to give you a deeper understanding of the language and principles at work.
If you just want som code to copy-past and understand it while you got refer to section 7.

**Content**
1\. Introduction
2\. Now what is JavaScript
2.1 Everything is an object
3\. Creating an object
4\. Properties
5\. Functions
6\. Private properties
7\. Inheritance
8\. Conslusion

## <span style="color: #008000;">1\. Introduction</span>

If you are familiar with programming you will know that there are two major concepts to use when writing programs. One is called Precedural Programming. In this concept you write ordered functions in a top down design that take in and pass around your data. The other is called object oriented programming. Here you create objects, defined by classes and try to create a model of the real world using them. In this scenario your functions are modeled around, and often times attached to, your data. This is a very fast rundown, but basically this is how it works.

&nbsp;

## <span style="color: #008000;">2\. Now what is JavaScript?</span>

It looks like a procedural language, you only write function and code and there is no special built in class constructor syntax. In fact however JavaScript is object oriented. It works different than most object oriented languages, like Java or C# in the way that you do not have to define classes.

For the rest of this Tutorial I will assume that you are familiar with the principles and ideas behind object oriented programming.

### <span style="color: #339966;">2.1Everything is an object</span>

In JavaScript there is an object called "Object" and everything inherits from it. Every array, every object you create, and even functions (which are in fact objects) inherit from this fundamental object. In JavaScript you create pseudo:classes and then instances of them by linking objects with inheritance. This pseudo:class is also an object and it is called the "prototype".

For the rest of this tutorial I will help you become more familiar with object oriented programming in JavaScript.

&nbsp;

## <span style="color: #008000;">3\. Creating an object</span>

The most fundamental operation of cause is to create an object. In most languages you would now write a class and than create an instance of it. The most common way in JavaScript however is to create a constructor function.
<pre class="EnlighterJSRAW" data-enlighter-language="js">//Define a constructor function
function Dice(color) {
    this.color = color;
}

//Create an actual object
var dice = new Dice("blue");</pre>
&nbsp;

We just created a dice that has a blue color. As you can see it is not that different from writing a class and then creating an instance of it. Now lets take a look at the fundamentals of objects: properties and function.

&nbsp;

## <span style="color: #008000;">4\. Properties</span>

Properties are what is used by an object to store its state. An example of this is the color of our dice above. The above example already illustrates how to add properties to our object. Within the constructor function we simply call
<pre class="EnlighterJSRAW" data-enlighter-language="js">this.property [color=#FF4500]=[/color] value;</pre>
To associate a property of our object with a value. Lets add base to our dice that will be used to calculate the result of a role later.
<pre class="EnlighterJSRAW" data-enlighter-language="js">function Dice(color) {
    this.color = color;
    this.base = 6             // Just your every day dice.
}</pre>
&nbsp;

## <span style="color: #008000;">5\. Functions</span>

Another basic element of objects are function. They are used to do things, for example a dice may be rolled to get a result between 1 and the base of the dice. There are a few ways to do this which you commonly see used.
<pre class="EnlighterJSRAW" data-enlighter-language="js">//Common version one
function Dice(color) {
    this.color = color;
    this.base = 6;
    this.role = function() {
        return Math.floor((Math.random() * this.base) + 1);
    }
}

//Common version two
function Dice(color) {
    this.color = color;
    this.base = 6;
    this.role = role;
}

function role() {
    return Math.floor((Math.random() * this.base) + 1);
}</pre>
Both of these will work. But they are not the best way to do it. Let me show you how else you can do it and than I will explain what is happening here and when to use it.
<pre class="EnlighterJSRAW" data-enlighter-language="js">//Adding functions using prototype.
function Dice(color) {
    this.color = color;
    this.base = 6;
}

dice.prototype.role = function() {
    return Math.floor((Math.random() * this.base) + 1);
}</pre>
Here I used the prototype property which every function in JavaScript has. The prototype is what is used to create an object. More precisely if you call object.aFunction JavaScript will look up aFunction in the object. If it does not find the function defined there it will look up aFunction in object.prototype.
This has several advantages over the first two ways to do this. Using prototype you only add the function once while using the first two ways you add the function to each object that is created thus saving memory. Using prototype is closer to other object oriented languages, think about the prototype as the class of the object.
Use the prototype to add function or property to objects that you want to be the same for all instances of the object. Our role() is an example of this. Do not use the prototype to define object specific variables, like our color.

&nbsp;

## <span style="color: #008000;">6\. Private Properties</span>

Most object oriented languages provide a way to define who can alter properties. This can be very useful and powerful. The keyword here are "private", "public", and so on. In JavaScript it seems like everything is always public. Using the above ways to add properties to an object and in JavaScript functions and objects are basically the same, everyone can edit them after you create the object. Someone could for example change the base of our dice and role an eight.

But fear not, there is a way to actually make variables private. Using closures or namespacing. (for now I hope you know what that is, I may write a tut on this later)
Consider the following code of a dice that can be created with a base.
<pre class="EnlighterJSRAW" data-enlighter-language="js">function Dice(color, base) {
    this.color = color;
    this.role = function() {
        //Note how I reverence base here since we are with in a closure.
        return Math.floor((Math.random() * base) + 1);
    }
}</pre>
Due to using closures we can simply use the base that is given at the start to calculate the return of a role. This does however break the principle of using prototypes to add function to your objects.

&nbsp;

## <span style="color: #008000;">7\. Inheritance</span>

Another principle of OOP is Inheritance. The idea is to spcify a class and have classes extend it. So for example we have a dice and every dice can be roled. But maybe we have six-sided dices and four-sided dices. We can reflect this in our code by creating a classes for each type of dice that extend the main dice.
To do this in JavaScript we need to call the constructor of the object we want to extend in our new class and tell it that the function calling it will be "this" (a keyword used to set context). We should also make sure to extend the prototype correctly. Simply follow the example below.
<pre class="EnlighterJSRAW" data-enlighter-language="js">//Function to create a dice.
function Dice(color, base) {
    this.color = color;
    this.base = base;
}

//Add a method to the dice.
dice.prototype.role = function() {
    return Math.floor((Math.random() * this.base) + 1);
}

//A four-sided dice
function fourDice(color) {
    dice.call(this, color, 4);
}

//Make sure we use the correct prototype.
fourDice.prototype = Object.create(dice.prototype);</pre>
This code makes sure the look up for functions is done correctly. When a function on a fourDice is called, it will first look in the fourDice, than in it's prototype and after that in the prototype of dice.

&nbsp;

## <span style="color: #008000;">8\. Conclusion</span>

Object-oriented Programming in JavaScript works different than in most other languages. We use prototypes instead of classes. Once you grasp that difference though it should be basically the same.

&nbsp;

Now you know how to work Object-oriented in JavaScript, go out and code away.