---
layout: post
title: "True Hash Maps in JavaScript"
date: 2015-05-21
---

Using an object literal as a simple means to storing key-value pairs is common place within JavaScript. However, an object literal is not a true hash map and therefore poses potential liabilities if used in the wrong manner. While JavaScript may not offer native hash maps (at least not cross-browser), there is a superior alternative to object literals to capture the desired functionality without the pitfalls.

## The Problem with Object Literals

The problem resides in the prototype chain of object literals as the [properties and methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/prototype#Properties) inherited from the `Object` prototype can clash with the mechanisms used to assert the existence of a key. Take for instance the `toString` method, checking the existence of a key with the same name using the `in` operator will result in a false positive:

<div class="code-block">

<pre class="prettyprint lang-javascript">var map = {};
'toString' in map; // true
</pre>

</div>

This happens because the `in` operator looks up the object's prototype chain to find inherited properties, in this case the `toString` method. To resolve this, the `hasOwnProperty` method was conceived which only looks for the existence of a specified key as a direct property of an object:

<div class="code-block">

<pre class="prettyprint lang-javascript">var map = {};
map.hasOwnProperty('toString'); // false
</pre>

</div>

This works fine until you encounter a key named "hasOwnProperty". Overwriting this method would cause further attempts to invoke the `hasOwnProperty` method to result in unexpected behavior, most likely an error depending on the new value:

<div class="code-block">

<pre class="prettyprint lang-javascript">var map = {};
map.hasOwnProperty = 'foo';
map.hasOwnProperty('hasOwnproperty'); // TypeError
</pre>

</div>

A quick fix for this is leveraging a generic object literal, one that hasn't been tampered with, and executing it's `hasOwnProperty` method in the context of your hash map:

<div class="code-block">

<pre class="prettyprint lang-javascript">var map = {};
map.hasOwnProperty = 'foo';
{}.hasOwnProperty.call(map, 'hasOwnproperty'); // true
</pre>

</div>

This works without any issues, but nevertheless, the object still imposes restrictions on how it can be utilized. For instance, every time you wanted to enumerate the properties of the object within a `for ... in` loop, you would need to filter out the properties of the prototype chain:

<div class="code-block">

<pre class="prettyprint lang-javascript">var map = {};
var has = {}.hasOwnProperty;

for(var key in map){
    if(has.call(map, key)){
        // do something
    }
}
</pre>

</div>

This can become a bit tedious after awhile. Thankfully, there is a better way.

## Bare Objects

The secret to creating a true hash map is loosing the prototype all together, and the baggage that comes with it. To do this we can take advantage of the `Object.create` method [introduced in ES5](http://www.ecma-international.org/ecma-262/5.1/#sec-15.2.3.5). What's unique about this method is that you can explicitly define the prototype for a new object. For example, defining a plain object literal with a little more verbosity:

<div class="code-block">

<pre class="prettyprint lang-javascript">var obj = {};
// is equivalent to:
var obj = Object.create(Object.prototype);
</pre>

</div>

In addition to defining a prototype of your choosing, you can also forgo a prototype completely by passing a `null` value in place of a prototype object:

<div class="code-block">

<pre class="prettyprint lang-javascript">var map = Object.create(null);

map instanceof Object; // false
Object.prototype.isPrototypeOf(map); // false
Object.getPrototypeOf(map); // null
</pre>

</div>

These _bare_ (aka _dictionary_) objects are ideal for hash maps because the absence of a `[[Prototype]]` removes the risk of name conflicts. Since the object is completely void of any methods or properties, it will resist any type of coercion, attempting to do so would result in an error:

<div class="code-block">

<pre class="prettyprint lang-javascript">var map = Object.create(null);
map + ""; // TypeError: Cannot convert object to primitive value
</pre>

</div>

There is no primitive value or string representation because bare objects are not intended to be used for anything other than a key-value store, plain and simple. This serves to rationalize its distinct purpose. Keep in mind that the `hasOwnProperty` method is also lost to the bare object, which is okay because the `in` operator now works without any exceptions:

<div class="code-block">

<pre class="prettyprint lang-javascript">var map = Object.create(null);
'toString' in map; // false
</pre>

</div>

Better yet, those tedious `for ... in` loops become much simpler. We can finally write a loop the way it was meant to be:

<div class="code-block">

<pre class="prettyprint lang-javascript">var map = Object.create(null);

for(var key in map){
    // do something
}
</pre>

</div>

Despite the differences, for all intents and purposes, it still behaves much like an object literal. Properties can be accessed using dot or bracket notation, the object can be stringified, and the object can still be employed as the context of a method from the `Object` prototype:

<div class="code-block">

<pre class="prettyprint lang-javascript">var map = Object.create(null);

Object.defineProperties(map, {
    'foo': {
        value: 1,
        enumerable: true
    },
    'bar': {
        value: 2,
        enumerable: false
    }
});

map.foo; // 1
map['bar']; // 2

JSON.stringify(map); // {"foo":1}

{}.hasOwnProperty.call(map, 'foo'); // true
{}.propertyIsEnumerable.call(map, 'bar'); // false
</pre>

</div>

Even different methods of type checking will indicate what you would expect from an object literal:

<div class="code-block">

<pre class="prettyprint lang-javascript">var map = Object.create(null);

typeof map; // object
{}.toString.call(map); // [object Object]
{}.valueOf.call(map); // Object {}
</pre>

</div>

All this makes it simple to replace object literals with bare objects and have them integrate nicely into a pre-existing application without forcing wide spread changes.

## Conclusion

In the context of simple key-value stores, a bare object is the clear successor to object literals, helping to alleviate the quirks with a clearly defined purpose. For more fully-featured data structures, ES6 will introduce native hash maps in the form of the [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) and [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set) objects, among others. Until then, or even after, you should look to bare objects for all your basic hash map needs.