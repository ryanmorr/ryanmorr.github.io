---
layout: post
title: Using Mutation Observers to Watch for Element Availability
date: 2015-03-16
project: https://github.com/ryanmorr/ready
tags:
  - DOM
  - JavaScript
---

Many developers have become accustomed to using some variant of a DOM ready method to signify when they can begin to traverse and manipulate the DOM. Or, better yet, placing the scripts at the bottom of the HTML to avoid blocking render while ensuring the DOM is loaded when the JavaScript is executed. However, in this day and age of dynamic web applications, it seems like a rather antiquated approach. The DOM is not static, it is alive and subject to manipulation long after the initial page load. Simply knowing when the DOM is ready, while still useful, does not encompass the entirely of the desired functionality. Instead, how about watching for when specific elements become available throughout the course of runtime? Well, I intend to offer a solution to support exactly that kind of functionality using [mutation observers](https://developer.mozilla.org/en/docs/Web/API/MutationObserver) as the means to a more modern approach.

## Mutation Observers

The [W3C DOM4 specification](http://www.w3.org/TR/dom/#mutation-observers) initially introduced mutation observers as a replacement for the deprecated [mutation events](https://developer.mozilla.org/en-US/docs/Web/Guide/Events/Mutation_events). The new specification essentially permits the same functionality, but with some added benefits. Most notably is that mutation observers are asynchronous, and will invoke the callback after a batch of changes rather than every single change. This allows mutation observers to operate in a more performant manner, one of the pitfalls of mutation events, [among others](http://lists.w3.org/Archives/Public/public-webapps/2011JulSep/0779.html).

Mutations that can be observed include the addition and removal of child nodes, changes to attributes, and updates to text nodes. For the purposes of this article, we want to focus on the addition of elements into the DOM. For this, we need to create a new instance of `MutationObserver` passing the callback function as the only argument. Then we invoke the `observe` method with the element we want to be observed and a configuration object as the first and second arguments:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
var observer = new MutationObserver(callback);
observer.observe(document.documentElement, {
    childList: true,
    subtree: true            
});
</pre>
</div>

The `childList` configuration option causes the observer to monitor the DOM for any nodes added or removed from the `documentElement`. The `subtree` option will additionally allow every child node of the `documentElement` to be monitored for these same changes. This gives us the means to detecting element insertion anywhere in the DOM, which bodes well for our purposes. The next step is to formulate the solution.

## Watching for Desired Element Availability

The solution supports detecting element readiness on the initial page load as well as elements dynamically appended to the DOM. The code in full is as follows:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
(function(win) {
    'use strict';
    
    var listeners = [], 
    doc = win.document, 
    MutationObserver = win.MutationObserver || win.WebKitMutationObserver,
    observer;
    
    function ready(selector, fn) {
        // Store the selector and callback to be monitored
        listeners.push({
            selector: selector,
            fn: fn
        });
        if (!observer) {
            // Watch for changes in the document
            observer = new MutationObserver(check);
            observer.observe(doc.documentElement, {
                childList: true,
                subtree: true
            });
        }
        // Check if the element is currently in the DOM
        check();
    }
        
    function check() {
        // Check the DOM for elements matching a stored selector
        for (var i = 0, len = listeners.length, listener, elements; i &lt; len; i++) {
            listener = listeners[i];
            // Query for elements matching the specified selector
            elements = doc.querySelectorAll(listener.selector);
            for (var j = 0, jLen = elements.length, element; j &lt; jLen; j++) {
                element = elements[j];
                // Make sure the callback isn't invoked with the 
                // same element more than once
                if (!element.ready) {
                    element.ready = true;
                    // Invoke the callback with the element
                    listener.fn.call(element, element);
                }
            }
        }
    }

    // Expose `ready`
    win.ready = ready;
            
})(this);
</pre>
</div>

The `ready` function accepts a selector string as the first argument and the callback function as the second. Once ready, the callback is invoked, passing the element as the only parameter:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
ready('.foo', function(element) {
    // do something
});
</pre>
</div>

What is important to note here is that a `ready` invocation is not a one-off binding. Instead, the callback lives throughout the life of the page, and is invoked any time an element matching the selector string is added to the DOM. If multiple elements matching the selector are added at once, the callback is invoked in succession for each element in document order.

## Conclusion

This serves as a more robust and forward-thinking approach to initializing DOM traversal and manipulation than your typical DOM ready methods. Despite growing [browser support for mutation observers](http://caniuse.com/#feat=mutationobserver), some browsers such as IE 10 and below lack support. While it may be too early for widespread adoption, it could be a small window into the future of DOM initialization via JavaScript. View full documentation and download the source for this project on [GitHub](https://github.com/ryanmorr/ready).