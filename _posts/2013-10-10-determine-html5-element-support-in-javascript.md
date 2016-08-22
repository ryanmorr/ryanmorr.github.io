---
layout: post
title: Determine HTML5 Element Support in JavaScript
date: 2013-10-10
categories:
  - Articles
  - Projects
tags:
  - Feature Testing
  - HTML
  - HTML5
  - JavaScript
---

The dawn of HTML5 brought about a whole bunch of new elments. However, like CSS3, the specification is still relatively new and many of the browsers are slow to adopt the new elements. Elements such as `<header>`, `<footer>`, `<nav>`, `<section>`, `<progress>`, `<template>`, `<aside>`, `<article>`, and `<canvas>` do not possess widespread browser support just yet. Despite this level of inconsistent support, you should not be deterred from using these new tags, but it may be pertinent to be aware of their availability.

## HTML Interfaces

It is the [standardization](http://www.w3.org/TR/DOM-Level-2-HTML/html.html#ID-011100101) of the `HTMLElement` object interface and the inherent hierarchy that makes it possible to reliably resolve an element's type and therefore support. Included in the hierarchy is the `HTMLUnknownElement` interface which acts as a catch-all for invalid HTML elements. In other words, any element you define for which there is no associated interface in the `HTMLElement` hierarchy, will default to an instance of the generic `HTMLUnknownElement` interface. This is true whether you create the element via `document.createElement` or explicitly write it in your markup. This means we can leverage `Object.prototype.toString` to divulge an element's internal `[[Class]]` property to ascertain if it is an unknown element:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
function isElementSupported(tag){
    var element = document.createElement(tag);
    return Object.prototype.toString.call(element) !== '[object HTMLUnknownElement]';
}
</pre>
</div>

Browser support for the interface is good as well, according to [MDN](https://developer.mozilla.org/en/docs/Web/API/HTMLUnknownElement), the `HTMLUnknownElement` interface is implemented on all the stock desktop and mobile browsers. Unfortunately, it seems IE8 and less does not support this, instead implementing a `HTMLGenericElement` interface.

Bear in mind that it may still be prudent to employ more complicated feature tests to determine support for the actual functionality of an element, such as the case with support for specific formats and codecs of the video and audio elements. 

## Inconsistencies

Unfortunately, there are [inconsistencies](http://kangax.github.io/jstests/html5_elements_interfaces_test/) with some elements and their associated interfaces. Some elements that are actually supported (or at least reported to be), inherit from the `HTMLUnknownElement` interface. For example, according to [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/time#Browser_compatibility) and [caniuse.com](http://caniuse.com/#feat=html5semantic), the `time` element has been supported in Chrome since version 33. Yet its internal `[[Class]]` property points to the `HTMLUnknownElement` interface instead of the `HTMLTimeElement` interface as it should. As such, the `datetime` attribute inherent to the `time` element doesn't exist:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
'dateTime' in document.querySelector('time'); // false
</pre>
</div>

All indications would suggest the `time` element does not exist in Chrome, despite what is being reported. In fact, even though the `HTMLTimeElement` interface is clearly defined within the [specification](http://www.w3.org/html/wg/drafts/html/master/#the-time-element), only Firefox 22+ has actually implemented it thus far. Opera had included it in version 11.50 only to remove it in version 15. It would seem the `time` element is in a state of limbo as different browsers decide how they want to implement it. Because of this, it may be too soon for the method above. However, moving forward, I would expect browsers to adhere to the specification and include the `HTMLTimeElement` interface.

## Conclusion

Thanks to HTML interfaces, we can now learn from where elements are derived and the capabilities they inherit. Despite a few false negatives, I would expect this method to be a reliable solution for determining element support in the future, eliminating less reliable element inferences. You can see how your browser stacks up by viewing this [CodePen](http://codepen.io/ryanmorr/pen/EaWROJ). For the latest version of this code, please refer to the project on [GitHub](https://github.com/ryanmorr/is-element-supported).