---
layout: post
title: Detecting CSS Style Support
date: 2013-08-21
project: https://github.com/ryanmorr/is-style-supported
categories:
  - Articles
  - Projects
tags:
  - CSS
  - Feature Testing
  - JavaScript
---

CSS tends to be in a constant phase of transition as new specifications are continuously proposed, drafted, and then left to the browsers for implementation. How and when a new feature is implemented is determined by the browser, often including their vendor prefix (-moz-, -webkit-, -o-, -ms-) to further dilute the feature. In fact, sometimes the W3C will define an official specification for a feature after one or more browsers have already implemented it. Despite the emergence of CSS3 in both support and usage over the last couple years, it is still very much in the early stages of standardization and implementation which is often changing and debated over. To help combat the confusion, the following article will focus on methods of determining support not just for styles but also their supported assignable values.

## Feature Queries

To help contend with this very problem, the W3C has drafted a new specification for an `@supports` rule. It's used similarly to an if statement by providing a condition and nested CSS statements that are only rendered if the condition is met:

<div class="code-block">
  <pre class="prettyprint lang-css">
@supports (display: flex) {
    #content {
        display: flex;
    }
}
</pre>
</div>

There is also an equivalent JavaScript API called `CSS.supports` or `supportsCSS` in Opera 12.1. This function supports two methods of usage by accepting either a property and value as separate arguments or a single conditional statement as the only argument:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
CSS.supports('transform-origin', '5px');
CSS.supports('(display: table-cell) and (display: list-item)');
</pre>
</div>

It is encouraging to know that the W3C seems to have acknowledged the discrepancies between browsers and their varying levels of CSS style support. Unfortunately the `@supports` rule is still very much in the early stages of adoption and is currently only supported in Firefox 22+, Chrome 28+, and Opera 12.1+. You can checkout the new API on [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/@supports) to read more. 

## Feature Testing Styles

Due to the lack of support for the `@supports`, we'll require a JavaScript-based fallback solution to mimic its capabilities. Since browser detection is entirely unreliable and unnecessary, we need to find something better. Instead we will employ feature testing to ascertain the browser's capability of computing the property and value we are trying to determine support for. The idea here is simple, if the browser can interpret the style than it must be supported, otherwise it must not.

For properties, examining the `style` object of an element and ensuring that the property exists will reliably resolve support in all mainstream browsers:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
animationName in element.style;
transform in element.style;
</pre>
</div>

That was easy, but to feature test support for assignable property values we need to get a little more creative. Manipulating an element's `style` object is unreliable as it will have no effect for unsupported declarations and instead will behave like a simple object literal property-value assignment. Alternatively, adding the property and value as an inline style using an element's `style.cssText` property will invoke the CSS interpreter to determine support. Then inspecting the `style` object will reveal that unsupported property values will always return an empty string:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
element.style.cssText = "color:foo; width:10px";
element.style.color !== ""; // false
element.style.width !== ""; // true
</pre>
</div>

Now that we can determine support for properties and their values, we can begin to formulate a solution.

## The Solution

Throwing these techniques together while utilizing native implementations when available gives us the following function:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
function isStyleSupported(prop, value) {
    // If no value is supplied, use "inherit"
    value = arguments.length === 2 ? value : 'inherit';
    // Try the native standard method first
    if ('CSS' in window && 'supports' in window.CSS) {
        return window.CSS.supports(prop, value);
    }
    // Check Opera's native method
    if('supportsCSS' in window){
        return window.supportsCSS(prop, value);
    }
    // Convert to camel-case for DOM interactions
    var camel = prop.replace(/-([a-z]|[0-9])/ig, function(all, letter) {
        return (letter + '').toUpperCase();                          
    });
    // Check if the property is supported
    var support = (camel in el.style);
    // Create test element
    var el = document.createElement('div');
    // Assign the property and value to invoke
    // the CSS interpreter
    el.style.cssText = prop + ':' + value;
    // Ensure both the property and value are
    // supported and return
    return support && (el.style[camel] !== '');
}
</pre>
</div>

The `isStyleSupported` function accepts a style property as the first parameter in standard CSS notation (kebab-case or hyphenated format) and accepts the property value as an optional second argument. If no value is supplied, it will default to _inherit_ as it is supported by all properties. 

## Leveraging Supported Styles

Despite the fact that this method proves to be a reliable polyfill for JavaScript, we still have no means of creating a substitute for the `@supports` rule in CSS. However, by adding classes to the `<html>` element that represent support for properties/values, we can utilize them in our stylesheets:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
if(isStyleSupported('animation-name')){
    document.documentElement.classList.add('css-animation');
}

if(isStyleSupported('display', 'flex')){    
    document.documentElement.classList.add('flex-display');
}
</pre>
</div>

Then we can create style declarations such as the following to take advantage of the feature in supporting browsers:

<div class="code-block">
  <pre class="prettyprint lang-css">
.css-animation #dialog {
    animation-name: popup;
    animation-duration: 1s;
}

.flex-display .column {
    display: flex;  
}
</pre>
</div>

This type of solution is not ideal because of its obtrusive nature, violating the separation of concerns, but it's a solution nonetheless.

## Conclusion

With the help of this method we take advantage of new styles now, leveraging progressive enhancement when possible until support for the `@supports` API increases. For the latest version of this code, please refer to the project on [GitHub](https://github.com/ryanmorr/is-style-supported).