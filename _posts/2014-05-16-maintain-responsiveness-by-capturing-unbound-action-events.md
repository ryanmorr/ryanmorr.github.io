---
layout: post
title: Maintain Responsiveness by Capturing Unbound Action Events
date: 2014-05-16
project: https://github.com/ryanmorr/action-observer
categories:
  - Articles
  - Projects
tags:
  - DOM
  - Event
  - JavaScript
  - UX
---

Responsiveness is critical for modern client-side applications. When a user clicks on something, they expect a result and some are not too keen on waiting. At the very least, a user&#8217;s actions should be acknowledged via a loading indicator or the like. The absolute worse case scenario is having nothing happen at all. This is often the case pre-initialization. The user interacts with a component without any effect or even error message because, unbeknownst to the user, the page is still in the process of initialization and event handlers had yet to be bound to their correlative elements. It&#8217;s an issue I have encountered on numerous occasions. A less technically inclined user may assume the site is broken, that makes for a bad first impression. For this article, I will be exploring a solution to this problem by way of capturing user triggered action events pre-initialization to maintain responsiveness.

## The Concept

Typically, progressive enhancement is the obvious answer to this problem and should still be instituted as a bulletproof fallback. However, if you wish to serve a highly dynamic application and want to limit page reloads or avoid them all together, than it may not be the ideal solution.

Progressive enhancement aside, there are two viable approaches you can take to resolve the problem. You could present the user with a loading mask until initialization has completed to inhibit interaction with the interface, such is the case with GMail. Or you can capture the action events, such as clicks and form submissions. The latter can be achieved by utilizing event delegation to listen for user triggered events that bubble up to the document element. This way the event can be handled immediately or queued for processing once the page has finished initializing while still acknowledging the user&#8217;s actions. All that is required is a means of distinguishing between events that should be intercepted by JavaScript and those that should not. It wouldn&#8217;t be to practical to prevent a hyperlink from following the link that otherwise shouldn&#8217;t be interrupted. 

## Introducing ActionOberver

The following solution, I&#8217;ve dubbed `ActionObserver`, observes click events and form submissions to capture user interactions. It is a simplified solution that could be easily expanded to include support for key events to detect user keystrokes, touch events to detect interactivity, or scroll events to handle dynamic progressive loading. The code in its most basic form looks like the following:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
(function(win){ 
    'use strict';

    var ActionObserver = {},
    docElement = win.document.documentElement, 
    listeners = {};

    // Listen for action events on the document when they bubble up
    docElement.addEventListener('click', onEvent, false);
    docElement.addEventListener('submit', onEvent, false);

    // Handle action events (click, submit) on the document
    function onEvent(e){
        // Find the first ancestor element of the event target
        // containing the data-observe attribute
        var el = e.target.closest('[data-observe]'), key;
        if(el){
            if(el.nodeName.toLowerCase() === 'form' && e.type !== 'submit'){
                return;
            }
            // Get the value of the data-observe attribute
            // to find the callback function and invoke it
            key = el.getAttribute('data-observe');
            if(listeners.hasOwnProperty(key)){
                listeners[key](e, el);
            }
        }
    }

    // Map a callback function to the element in 
    // which you would like to observe action events on
    ActionObserver.bind = function(key, fn, ctx){
        listeners[key] = fn.bind(ctx || win);
    };

    win.ActionObserver = ActionObserver;

})(this);
</pre>
</div>

Browser support is good as well, only IE 8 and less does not support the bubbling of submit events to the document and therefore cannot be detected and captured.

## Usage

To start, add a `data-observe` attribute to any element on which you want to listen for action events. This helps to differentiate which events should be intercepted by `ActionObserver` and which should not. For example:

<div class="code-block">
  <pre class="prettyprint lang-html">
&lt;!--Observe click events--&gt;
&lt;a href="#" data-observe="add"&gt;Add Item&lt;/a&gt;

&lt;!--Observe submit events--&gt;
&lt;form method="GET" action="#" data-observe="search"&gt;
    &lt;input type="search" name="search" /&gt;
    &lt;button type="submit"&gt;Submit&lt;/button&gt;
&lt;/form&gt;
</pre>
</div>

Adding the `data-observe` attribute to a form will automatically capture submit events for that form. Adding the `data-observe` attribute to any other type of element will automatically observe click events.

The value of the `data-observe` attribute is the reference point in JavaScript to that element. Use the `ActionObserver.bind` method to attach a callback function that will be invoked when an action event is dispatched from the source element, passing the event object and the element as arguments:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
ActionObserver.bind('add', function(event, element){
    // Process or queue request                     
});

ActionObserver.bind('search', function(event, element){
    // Process or queue request                     
});
</pre>
</div>

Ideally, you should inline the code in the head of the document for JavaScript rich applications. This will block rendering, but you can avoid the extra HTTP request and get instant notification of user requests before the DOM has finished loading or scripts have finished initializing. So it&#8217;s a bit of a trade-off. Check out the project on [GitHub](https://github.com/ryanmorr/action-observer) for more information and the latest version of the code.

## Lazy Loading & Instantiation

This solution is ideal for lazy loading and instantiation of components that are not immediately required for page load. If your a proponent of asynchronous module loading, you could use RequireJS to load the dependencies of a component once it is requested by the user. Take for instance the following:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
ActionObserver.bind('search', function(event, element){
    require(['search', 'text!search.tpl'], function(search, template){
        // Initialize component and process request          
    });                     
});
</pre>
</div>

This approach allows you to limit the amount of HTTP requests for the initial page load to the bare essentials. Only when specific functionality is requested by the user via their interactions is the required resources automatically loaded and initialized.

## Conclusion

Employing this technique allows you to place all your JavaScript at the bottom of the page to leverage increased rendering time and usability (specifically for above the fold content) without sacrificing responsiveness for JavaScript-based functionality. This makes for a user experience that is unobtrusive, consistent, but most importantly, responsive.