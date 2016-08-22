---
layout: post
title: Leveraging the Most Out of CSS Classes
date: 2013-09-01
categories:
  - Articles
tags:
  - CSS
  - HTML
---

To harness the most out of CSS classes is to take advantage of what truly is the unsung hero of advanced RIA development as it relates to the presentation layer. When we observe our markup, we find that various portions of the DOM may require styling influences on the initial page load as well as interaction events or cues to adjust its structure, positioning, visibility, state, and skin. For this, classes are the clear suitor as it brings flexibility, modularity, inheritance, and a dynamic and unobtrusive nature to the table.

## Benefits

One benefit in particular is in an element's ability to support multiple class declarations at a time which, in a sense, facilitates a multiple inheritance of sorts. This gives us the unique ability within CSS to separate an element's various states and differing characteristics to create a more modular approach in our stylesheets. By doing so, we can create classes that are in no way specific to any one element but merely represent a characteristic or group of characteristics that any element can adopt provided the class name.

Inheritance can also play a key role in the design process, by letting the cascade flow through the DOM we can leverage the styles of ancestor elements and avoid a rigid implementation with redundant style declarations. If we employ this technique, we can easily alter state of descendant elements by adding or removing a class to or from that of an ancestor element while avoiding the need to traverse the DOM to manipulate each descendant element individually in JavaScript.

As far as client-side scripting is concerned, classes are the preferred window to CSS from JavaScript. By hiding styling characteristics behind class names, JavaScript can merely alter an element's `className` attribute to activate or deactivate a specific state of that element. In this case, JavaScript is not concerned with what styling influences are taking place on page load or even after event triggered changes in state. This allows us to quickly consult our CSS to make minor styling changes rather than JavaScript.

## Abstract Classes

Abstract classes are those that do not necessarily reflect a state or any specific element but common characteristics shared between elements of varying purpose and context. Take for instance the following:

<div class="code-block">
  <pre class="prettyprint lang-css">
.widget {
    font-family: Trebuchet MS, Tahoma, Verdana, Arial, sans-serif;
    font-size: 1.1em;
}

.overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}
</pre>
</div>

As you can see these examples are of a very generic nature both in name and styling influences as they represent a very high level abstraction. Typically classes such as these should always be combined with others as part of an element's class declaration to round out its structure, positioning, state, and skin.

The advantage of abstract classes do not come solely in their ability to apply styles to an element in a modular fashion, but also in the relationships they create. Think of it from the perspective of object-oriented programming, provided the overlay class, an element will inherit the role of an overlay. For example: 

<div class="code-block">
  <pre class="prettyprint lang-html">
&lt;div class="widget"&gt;
    &lt;div class="overlay widget"&gt;
        &lt;!-- widget markup --&gt;
    &lt;/div&gt;
&lt;/div&gt;
</pre>
</div>

In this case, the parent element _”is a”_ widget and _”has a”_ widget and _”has a”_ overlay while the child element _”is a”_ widget and _”is a”_ overlay. Although the child element is both an overlay and a widget, it is important to recognize that this element can be referenced by either statement together or independently of the other, like so:

<div class="code-block">
  <pre class="prettyprint lang-css">
.widget .overlay {
    /* target overlays that belong to widgets */
}

.widget .widget {
    /* target widgets that belong to widgets */
}

.widget .widget.overlay {
    /* target overlays that are also widgets that belong to widgets */
}
</pre>
</div>

By grouping elements together in this manner we can target elements based on their role, state, structure, positioning, document position, or relativity to other elements. From here we let inheritance fill in the gaps; as we move further down the DOM hierarchy, the more precise and relevant our styles become to the specific elements to which they are applied. Width and height, for example, should typically be reserved for styling in a low level context, while font (family, size, and color) is best served at a high level. Not to mention that this grouping makes it very easy for JavaScript to quickly reference or query for various elements in various contexts for fast and easy manipulation.

## Character Classes

Similar to abstract classes, character classes work on a lower level context and are designed to fulfil one purpose relating to a characteristic or group of characteristics. They help to achieve both visual and cross-browser consistency across an application in the purpose that they serve. For example, the following are some character classes of varying purposes:

<div class="code-block">
  <pre class="prettyprint lang-css">
.hidden {
    display: none;
}

.responsive {
    display: block;
    width: 100%;
    -webkit-box-sizing: border-box;
    -moz-box-sizing: border-box;
    box-sizing: border-box;
}

.center {
    margin-left: auto;
    margin-right: auto;
}
</pre>
</div>

Typically character classes do not need to be overridden in a lower level context as they should be mutually exclusive from your structure and skin. You'll also want to avoid depending on character classes to target descendants as part of a selector statement. Doing so will expand the scope of the class beyond its original purpose, this can snow ball into further dependencies that result in a loss of modularity and flexibility. 

Exploiting character classes to the fullest means easy adoption of shared markup conventions and allow for ease in code integration. By putting together a small component library, you will find that they will not only provide reusable classes throughout an entire application but also amongst projects.

## Themed Classes

As you might have guessed, themed classes have everything to do with your skin, this means fonts, colors, background images, formatting, icons, and states. Take for instance the following:

<div class="code-block">
  <pre class="prettyprint lang-css">
.loading {
    cursor: wait;
}

.active {
    border: 1px solid #7abcff;
    background: #4096ee;
    color: #ffffff;
}

.disabled {
    cursor: default;
    opacity: 0.6;
    pointer-events: none;
}
</pre>
</div>

Classes such as these are most useful when applying a theme to various user interface widgets. We can use a group of semantic presentation classes to indicate the state of a widget such as default, hover, active, and disabled. As the user interacts with the page, we dynamically apply the appropriate class to the widget to represent its current state. This level of class consistency makes it easy to ensure that all elements with a similar role or state will look the same across all widgets.

Like abstract classes, themed classes also create relationships, however these relationships are more targeted towards an element’s state as opposed to its role. This makes it simple to target elements based on their current state, for instance active elements:

<div class="code-block">
  <pre class="prettyprint lang-css">
.overlay.active .header {
    /* target the header of active overlays */
}

.overlay.active .body {
    /* target the body of active overlays */
}
</pre>
</div>

Here we can add additional rules to automatically style the descendants of an element based on its state without directly referencing the descendants in CSS via additional class names or in JavaScript via DOM traversal. 

Progressive enhancement also falls under the cloud of themed classes as it can be utilized to bring greater aesthetics to your skin. It differs in the sense that a class will be responsible for providing a single detail (rounded corners, drop shadow, gradient backgrounds, etc.) to the theme in a cross-browser and consistent manner:

<div class="code-block">
  <pre class="prettyprint lang-css">
.shadow {
    -webkit-box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.5);
    -moz-box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.5);
    box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.5);
}

.rounded-corners {
    -webkit-border-radius: 5px;
    -moz-border-radius: 5px;
    border-radius: 5px;
}
</pre>
</div>

By grouping vendor specific styles together (-webkit-, -moz-, -o-, -ms-) elements will adopt the behaviour in supported browsers and degrade gracefully in unsupported browsers. This helps to keep code not only backwards compatible but also future proof.

If properly implemented we can achieve visual consistency across an application and allow components to be theme-able in a rather simplified and modular fashion.

## Conclusion

When we combine all these techniques, we find utter simplicity in forming both the structure and skin of an element, while minimizing the amount of element specific style declarations in our stylesheet. If done right, you will find your markup will greatly improve as a result and it will be as easy to write as the following:

<div class="code-block">
  <pre class="prettyprint lang-html">
&lt;div class="widget overlay rounded-corners shadow active"&gt;
    &lt;div class="header"&gt;
        &lt;!-- header markup --&gt;
    &lt;/div&gt;
    &lt;div class="body"&gt;
        &lt;!-- body markup --&gt;
    &lt;/div&gt;
    &lt;div class="footer"&gt;
        &lt;!-- footer markup --&gt;
    &lt;/div&gt;
&lt;/div&gt;
</pre>
</div>

These techniques help to serve as an advantage not only to your CSS but also your markup and JavaScript for a truly unobtrusive implementation. Changing the theme of your application could be as easy as switching a stylesheet to provide a whole new look & feel.