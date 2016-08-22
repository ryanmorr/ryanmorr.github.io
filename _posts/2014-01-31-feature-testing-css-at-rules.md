---
layout: post
title: Feature Testing CSS At-Rules
date: 2014-01-31
project: https://github.com/ryanmorr/is-at-rule-supported
categories:
  - Articles
  - Projects
tags:
  - CSSOM
  - Feature Testing
  - JavaScript
---

The _CSS Object Model (CSSOM)_ is a [W3C specification](http://dev.w3.org/csswg/cssom/) that defines APIs for CSS in JavaScript, encompassing all of what CSS has to offer. This has provided JavaScript a new window into CSS and all of its supported features that was not previously available. This bodes well for feature testing. This article will be specifically addressing the detection of at-rules and how we can leverage them in supported browsers while still providing a fallback solution for those that don’t.

## The CSSRule Interface

Included in the CSSOM is the `CSSRule` [interface](https://developer.mozilla.org/en-US/docs/Web/API/CSSRule) which represents a single CSS at-rule. It acts as a high level abstraction for every type of CSS rule (charset, media, font-face, etc.), defining methods and properties common to all rule interfaces.

Among the properties is a set of static [type constants](http://wiki.csswg.org/spec/cssom-constants#cssom-constants) that corresponds with the more specialized interface specific to each type of rule. A rule instance&#8217;s type is contained in the `type` property that returns the associated type constant. These constants are what make feature testing rule support possible.

## Feature Testing At-Rules

To detect support for an at-rule, checking the existence of the associated type constant is all that is required. This infers the existence of the corresponding interface and therefore functionality. For example:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
// Is the @keyframes rule supported?
if('KEYFRAMES_RULE' in window.CSSRule){
    // Do something
}
// Is the @font-face rule supported?
if('FONT_FACE_RULE' in window.CSSRule){
    // Do something
}
// Is the @supports rule supported? (oh the irony)
if('SUPPORTS_RULE' in window.CSSRule){
    // Do something
}
</pre>
</div>

Unfortunately, like everything else, this approach does not escape the dark cloud of vendor prefixes. This means you will have to expand the detection process to account for that as well:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
if('KEYFRAMES_RULE' in window.CSSRule || 
   'MOZ_KEYFRAMES_RULE' in window.CSSRule || 
   'WEBKIT_KEYFRAMES_RULE' in window.CSSRule){
    // Do something
}
</pre>
</div>

An alternative approach would be to check for the existence of the rule specific interfaces themselves in the global object:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
// Does the keyframe interface exist?
if('CSSKeyframesRule' in window){
    // Do something
}
</pre>
</div>

It&#8217;s merely a matter of preference, both techniques are equally valid and will still require vendor prefixes when necessary.

The fact that the CSSOM is standardized bodes well for cross-browser consistency. As of today, all major browsers, with the exception of IE8 and less, support the CSS object model including the `CSSRule` interface and the type constants.

## A Complete Solution

We can simplify this approach even further by wrapping the solution in a function and offer a more usable API that doesn&#8217;t require the vendor prefixes or a pre-existing knowledge of the CSSOM. The code is as follows:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
function isAtRuleSupported(rule) {
    var support = false;
    // Create an array of vendor prefixes
    var prefixes = ['MOZ_', 'WEBKIT_', 'O_', 'MS_', ''], length = prefixes.length;
    // Convert the rule to a form compatible with the `CSSRule` type constants
    rule = rule.replace(/^@/, '').toUpperCase().split('-').join('_') + '_RULE';
    while (!support && length--) {
        // Starting with the unprefixed version, iterate over all the vendor
        // prefixes to determine browser support
        support = (prefixes[length] + rule) in window.CSSRule;
    }
    return support;
}
</pre>
</div>

Usage is fairly straightforward, invoking `isAtRuleSupported` with only a string of the rule name in standard CSS notation.

## Conclusion

Like most feature detects these days, this solution is about trying to employ new features where they are supported while still providing an alternative solution for non-supporting browsers to create a seamless experience. Progressive enhancement at its finest. You can find the latest version of this code on [GitHub](https://github.com/ryanmorr/is-at-rule-supported).