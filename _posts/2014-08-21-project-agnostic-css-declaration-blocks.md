---
layout: post
title: Project Agnostic CSS Declaration Blocks
date: 2014-08-21
tags:
  - CSS
---

Low specificity is key to good fundamental CSS design. Avoiding overly specific selectors and grouping similar characteristics helps to modularize your stylesheets resulting in greater portability and reusability of declaration blocks. This serves as an effective means to decoupling your CSS from your HTML. For this article, I will be focusing on some useful project agnostic declaration blocks, many of which form the basis of my CSS boilerplate given the ease in which they can be applied to any project without being too specific to any one project.

## Border Box

First [brought to light](http://www.paulirish.com/2012/box-sizing-border-box-ftw/) by Paul Irish and later [improved upon](http://blog.teamtreehouse.com/box-sizing-secret-simple-css-layouts#comment-50223) by Jonathan Neal, the following snippet helps to improve the default box model of the browser:

<div class="code-block">
  <pre class="prettyprint lang-css">
html {
    box-sizing: border-box;
}
*, *:before, *:after {
    box-sizing: inherit;
}
</pre>
</div>

Assigning the `box-sizing` property to _border-box_ allows the padding and border to be contained within the declared width and height of an element. This better facilitates responsive design. Percentages in particular become much more predictable and simpler to use.

By universally inheriting the `box-sizing` property from the `html` element, we can maintain the same result as the original code by Paul Irish while also preserving inheritance. This allows elements to acquire their box-model directory from its parent element, making it easier to change the `box-sizing` for plugins that use a different box-model.

## Clearfix

If your looking for the latest iteration of the _clearfix_, check out the following courtesy of [Nicolas Gallagher](http://nicolasgallagher.com/micro-clearfix-hack/). It helps with float-based layouts by applying the class to a container element, allowing the container to stretch to accommodate the floating elements consistently between browsers:

<div class="code-block">
  <pre class="prettyprint lang-css">
.clearfix:before,
.clearfix:after {
    content: " ";
    display: table;
}

.clearfix:after {
    clear: both;
}
</pre>
</div>

The value of _table_ rather than _block_ for the `display` property is necessary to prevent top-margin collapse of child elements when utilizing the `:before` pseudo-element. The `:after` pseudo-element is used to clear the floats. Additionally, it&#8217;s worth noting that the empty space assigned to the `content` property is not a typo and is instead required to resolve a bug associated with Opera.

## Positioning

The next few declaration blocks are fairly common classes used to aid in design by providing an easy means to positioning elements, helping to align them left, right, and center:

<div class="code-block">
  <pre class="prettyprint lang-css">
.pull-right {
    float: right !important;
}

.pull-left {
    float: left !important;
}

.center-x {
    display: block;
    margin-right: auto;
    margin-left: auto;
}

.center-xy {
    margin: 0 auto;
    display: table;
    position: relative;
    top: 50%;
    -webkit-transform: translateY(-50%);
    transform: translateY(-50%);
}
</pre>
</div>

The `center-xy` class in particular is a clever solution to center elements not just horizontally, but also vertically. This is a solution many developers have desired for quite some time now. Kudos to [Creative Punch](http://creative-punch.net/2014/01/center-vertically-horizontally-using-css3-transform/) for its discovery.

## Visibility

These helper classes, derived from the [HTML5 Boilerplate](https://github.com/h5bp/html5-boilerplate/blob/master/src/css/main.css#L116), assist in managing visibility to both users and screen readers, while also offering an easy means to controlling visibility programmatically:

<div class="code-block">
  <pre class="prettyprint lang-css">
.show {
    display: block !important;
}

.hide {
    display: none !important;
    visibility: hidden !important;
}

.invisible {
    visibility: hidden;
}

.hidden-accessible {
    border: 0;
    clip: rect(0 0 0 0);
    height: 1px;
    margin: -1px;
    overflow: hidden;
    padding: 0;
    position: absolute;
    width: 1px;
}
</pre>
</div>

The `hidden-accessible` class has the unique ability to mimic the effects of `display: none`, yet still be accessible to screen readers and web crawlers.

## Disabling Elements

The following declaration block sets the cursor to default, restricts mouse events, and fades the element into the background as a means of universally disabling anything from form fields to links to widgets in a consistent manner:

<div class="code-block">
  <pre class="prettyprint lang-css">
.disabled, 
:disabled,
[disabled] {
    cursor: default !important;
    opacity: 0.6;
    pointer-events: none;
    -moz-user-input: disabled;
    user-input: disabled;
    -moz-user-focus: ignore;
    user-focus: ignore;
}
</pre>
</div>

The `user-input` property is used to restrict user input into a field, while the `user-focus` property is used to restrain keyboard focus, skipping the element in the tab order. However, both properties do not possess wide-spread browser support (currently only supported in Firefox) and are still just proposals for inclusion to the specification.

## Image Replacement

The image replacement technique is used to visually hide descriptive text in favor of an image. This essentially allows you to provide a caption to an image available only to screen readers and web crawlers for better SEO:

<div class="code-block">
  <pre class="prettyprint lang-css">
.hide-text {
    text-indent: 100%;
    white-space: nowrap;
    overflow: hidden;
}
</pre>
</div>

The original technique set the `text-indent` property to -9999px in order to place the text well outside the viewport. However, the problem with this method was a performance hit caused by the browser drawing a giant 9999 pixel box off-screen.

The new technique discovered by [Scott Kellum](http://scottkellum.com/) and first written about at [Zeldman.com](http://www.zeldman.com/2012/03/01/replacing-the-9999px-hack-new-image-replacement/) addresses the performance issue. Instead, the text is simply indented beyond the width of its container, prevented from wrapping, and hides any overflow.

## Responsive Images

This class borrowed from [Bootstrap](https://github.com/twbs/bootstrap/blob/master/dist/css/bootstrap.css#L932) can make images responsive. This is ideal for mobile-friendly websites, allowing images to scale to fit its container element while still maintaining its aspect ratio:

<div class="code-block">
  <pre class="prettyprint lang-css">
.img-responsive {
    display: block;
    max-width: 100%;
    height: auto;
}
</pre>
</div>

The key to this class is setting the `max-width` property to _100%_. It allows the image to scale while also ensuring it does not scale larger than its original dimensions, preventing a reduction in the image&#8217;s quality.

## Transitions

Now that CSS3 transitions are widely supported across browsers, we can move simple animations from JavaScript to CSS where they belong. The following class provides an easy means to making any element completely animated by altering its properties in CSS or JavaScript:

<div class="code-block">
  <pre class="prettyprint lang-css">
.transition {
    -webkit-transition: all 0.3s ease-out;
    transition: all 0.3s ease-out;
}
</pre>
</div>

You can also use these helper classes adapted from [Bootstrap](https://github.com/twbs/bootstrap/blob/master/dist/css/bootstrap.css#L3170) for easy fading and collapsing/expanding of elements:

<div class="code-block">
  <pre class="prettyprint lang-css">
.fade {
    opacity: 0;
    -webkit-transition: opacity 0.15s linear;
    transition: opacity 0.15s linear;
}

.fade.in {
    opacity: 1;
}

.collapsing {
    position: relative;
    height: 0;
    overflow: hidden;
    -webkit-transition: height 0.35s ease;
    transition: height 0.35s ease;
}
</pre>
</div>

If your looking for a more comprehensive solution, you can check out [animate.css](https://github.com/daneden/animate.css) which provides animations for just about everything you can think of.

## Targeting Devices

Another set of declaration blocks adopted from [Bootstrap](https://github.com/twbs/bootstrap/blob/master/dist/css/bootstrap.css#L6116), these classes and media queries help to hide and show certain elements targeted at phones, tablets, and desktops:

<div class="code-block">
  <pre class="prettyprint lang-css">
.visible-phone, 
.visible-tablet, 
.visible-desktop {
    display: none !important;
}

.hidden-phone, 
.hidden-tablet, 
.hidden-desktop {
    display: block !important;
}

/* Phones */
@media screen and (max-width: 37.5em) {
    .visible-phone {
        display: block !important;
    }
    .visible-desktop, 
    .visible-tablet, 
    .hidden-phone {
        display: none !important;
    }
}

/* Tablets */
@media screen and (min-width: 37.5em) and (max-width: 64em) {
    .visible-tablet {
        display: block !important;
    }
    .visible-desktop, 
    .visible-phone, 
    .hidden-tablet {
        display: none !important;
    }
}

/* Desktops */
@media screen and (min-width: 64em) {
    .visible-desktop {
        display: block !important;
    }
    .visible-phone,
    .visible-tablet,
    .hidden-desktop {
        display: none !important;
    }
}
</pre>
</div>

I use _ems_ for the breakpoints to [better manage user zoom level](http://blog.cloudfour.com/the-ems-have-it-proportional-media-queries-ftw/) and prevent any awkwardly wrapped elements or other rendering issues. Exactly what you set your breakpoints to may depend on your project and supported devices.

## Conclusion

There you have it, a bunch of different helper classes and declaration blocks for a wide variety of purposes. This combined with some variant of a [CSS reset](http://www.cssreset.com/) for a consistent baseline and a responsive grid system for easy layout scaffolding, you will have a good starting point for any new project.