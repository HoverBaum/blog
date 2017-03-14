title: How to work without jQuery
tags:
  - jquery
  - js
id: 27
categories:
  - English
  - Webdev
date: 2014-07-06 13:29:22
---

**Content:**

*   The History of jQuery
*   What you can do without jQuery

    *   $ - Querying for elements
    *   .each - Work with elements from a list
    *   Manipulating elements

*   The big difference
*   Ease of use
*   The bottom line
&nbsp;

Many Webdesigners and developers today use jQuery for their website and web apps. The fact is: you probably don't need it.
To understand what you can do faster and better without jQuery let us first take a look at what jQuery is.
Once we have an understanding of jQuery I will go on to teach you how to replace many commonly used functions of jQuery with faster ones.

<!-- more -->

## <span style="color: #008000;">The History of jQuery</span>

In the dark ages of web development, about a few years back from now, it was a horrible ordeal to manipulate the DOM. This task however is one of the main reasons to use JavaScript. You make boxes bigger, things wiggle and add content to elements.
The problem was that every browser handled things bit differently. So you had to write different code for different browsers which no one really wants to do. To the rescue came once again a famous principle of programming: "Abstraction". Functions were created to do tasks and you could just call those functions, they would deal with handling different browsers. They also created wrappers for elements of the DOM so that would behave the same across browsers. Eventually this all combined into jQuery.

While not being the most accurate history of all times this illustrates why jQuery was created. You should take away from this that jQuery is an abstraction layer. As such it does tasks that JavaScript could do but makes them more easily accessible for you. This obviously results in an overhead since a function is called to call the function you want to use.

In modern times browsers have improved a great deal, so lets look at what you can do faster without jQuery.

## <span style="color: #008000;">What you can do without jQuery</span>

Keep in mind that all of this is for modern browsers, so if you are developing for IE7 or something it will not help you. For every day use and modern applications to run on up-to-date or semi-up-to-date browsers this will work.

### $ - Querying for elements

One of the most commonly used functions of jQuery is "$()". This bad boy does it all. It querys the DOM and returns elements and it creates wrappers for elements to work with.
Chances are, you are mainly using "$()" to query the DOM and get elements to manipulate. This is an easy to replace overhead, I present to you "document.querySelectorAll()". This does exactly the same thing. So the following two lines will return the same result.
<pre class="EnlighterJSRAW" data-enlighter-language="js">$(".myClass");
document.querySelectorAll(".myclass");</pre>
The only difference here is that the second method is faster.

Did I mention that queries can get even better?
<pre class="EnlighterJSRAW" data-enlighter-language="js">document.getElementsByClass("myClass");</pre>
This returns a HTMLCollection. A list of elements of the DOM that is automatically updated if the DOM changes. And it is also faster than "$()".

We just replaced one of the most used functions of jQuery with something faster. This is however not that pretty or easy to write code with. We will change that later, so keep reading.

### .each - Work with elements from a list

Another commonly used function of jQuery is .each(). Since this is a basic iterator we can use any loop to replace it. A nice way to do it would be like this.
<pre class="EnlighterJSRAW" data-enlighter-language="js">var elms = document.querySelectorAll(".myClass");
for (var e in elms) {
doSomething(e);
}</pre>

### Manipulating elements

Something we commonly want to do is to change things about elements. This may be, toggling, adding or removing a class, changing height/width, or setting the value of an input.

I will give a rundown of things that I consider commonly used for you to use as a reference.
**Editing Dimensions**
<pre class="EnlighterJSRAW" data-enlighter-language="js">//Change width of an element.
elm.style.width = "400px";
var height = 300;
elm.style.height = height + "px";
//jQuery
 $(elm).width(300);</pre>
**Setting the value of an input**
<pre class="EnlighterJSRAW" data-enlighter-language="js">//Set the value for an input
elm.value = "The value to set to";
//jQuery
$(elm).val("The value to set to");
</pre>
**Hide an element**
<pre class="EnlighterJSRAW" data-enlighter-language="js">//Hide an element
elm.style.display = "none";
//jQuery
$(elm).hide();</pre>
**Show an element that you hid**
<pre class="EnlighterJSRAW" data-enlighter-language="js">//Show a previously hidden element.
elm.style.display = "block"; //Or whatever you want to use.
//jQuery
$(elm).show();</pre>
**Create a new element**
<pre class="EnlighterJSRAW" data-enlighter-language="js">//Create a new element
var p = document.createElement("p");
var div = document.createElement("div");
//And so on...
//jQuery
var p = $("&lt;p&gt;&lt;/p&gt;");</pre>
**Append a created element to another**
<pre class="EnlighterJSRAW" data-enlighter-language="js">//Create element
var p = document.createElement("p");
elm.appendChild(p);
//jQuery
var p = $("&lt;p&gt;&lt;/p&gt;");
$(elm).append(p);</pre>
**Add a class**
<pre class="EnlighterJSRAW" data-enlighter-language="js">//Basic way to add a class.
elm.setAttribute("class", elm.className + " newClass");
//The " " (space) is important!
//jQuery
$(elm).addClass("newClass");</pre>
**Remove a class**
<pre class="EnlighterJSRAW" data-enlighter-language="js">//Basic removal of a class.
var newClassName = elm.className.replace("classToRemove", "");
elm.setAttribute("class", newClassName);
//jQuery
$(elm).removeClass("ClassToRemove");</pre>
**Toggle a class**
<pre class="EnlighterJSRAW" data-enlighter-language="js">//Toggle a class.
//Idea is to first check if it is there and than act accordingly.
if(elm.className.indexOf("classToToggle") &gt; -1) {
removeClass("classToToggle", elm);
//Assumed function that removes class form element since element has class.
} else {
addClass("classToToggle", elm);
//Again an assumed function. This time to add the class.
}
//jQuery
$(elm).toggleClass("classToToggle");</pre>
&nbsp;

## <span style="color: #008000;">The big difference</span>

Probably the biggest difference to get used to when moving away form jQuery is the way you call functions.
Using jQuery you are probably used to something like.
<pre class="EnlighterJSRAW" data-enlighter-language="js">$(Element).function();</pre>
You will have noticed that in the examples above I used.
<pre class="EnlighterJSRAW" data-enlighter-language="js">function(Element);</pre>
Using "$(Element)" created a jQuery-Wrapper of that element. This wrapper brings with is certain function you can call. As we have already seen many of these can be replaced by manipulating attributes of an element or have corresponding function that already exist. An example of the later is setting an attribute of an element of the DOM while an example of the first is setting the content on an element.
<pre class="EnlighterJSRAW" data-enlighter-language="js">//Set an attribute
//jQuery
$(img).attr("alt", "Description");
//JavaScript
img.setAttribute("alt", "Description");
//Set content
//jQuery
$(p).html("Your text here.");
//JavaScript
p.innerHTML = "Your text here.";</pre>
You may be used to something like this.
<pre class="EnlighterJSRAW" data-enlighter-language="js">//Add an onlclick listener.
$(elm).click(function() {
//When element is clicked add a class.
this.addClass("active");
});</pre>
If you like this syntax here is a similar way to do it with pure JavaScript.
<pre class="EnlighterJSRAW" data-enlighter-language="js">//Add an onclick listener.
elm.addEventListener("click", function() {
    //Return a function here since code is executed when the listener is added.
    return function() {
        //Here this is elm.
        this.style.height = "500px";
    }
}.call(elm));
//.call can be used to set "this" for the called function.
//The above is closeish to jQuery. It is however not really nice code. Just enables you to stick with the "this".
//I would recommend doing it a bit like the below.
elm.addEventListerner("click", function() {
//Here we only use the closure to bind "this" to "elm".
return handlerFunction.call(elm);
});
//While the function itself is somewhere else.
//This also allows to bind this handler to multiple objects, even in different loops over query returns.
function handlerFunction() {
this.style.height = "500px";
}
</pre>
&nbsp;

## <span style="color: #008000;">Ease of use </span>

A big point in using JavaScript instead of jQuery is ease of use. Actually more of the opposite. Typing "document.querySelectorAll()" every time you query instead of just "$()" is way longer.

As I mentioned above there is a solution.
jQuery is an overlay, so we will create our own one.
<pre class="EnlighterJSRAW" data-enlighter-language="js">//Define a function that calls the long one we do not want to type.
var get = function(query){ return document.querySelectorAll("query") };</pre>
Now we can type "get('.myClass')" and it will have the same effect as "$('.myClass')".
You could also use "$" for the function identifier instead of "get" and you would get jQuery with more performance.

&nbsp;

## <span style="color: #008000;">The bottom line</span>

Let me summarize what we learned.

jQuery is a neat overlay to make our lives easier.
If we want, we can use JavaScript functions instead of jQuery to improve performance.
We can also write shorter function that call the JavaScript function to make our life easier.

&nbsp;