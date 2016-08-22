---
layout: post
title: Abstract Away the Performance Faults of querySelectorAll
date: 2014-11-12
project: https://github.com/ryanmorr/query
tags:
  - DOM
  - JavaScript
  - Performance
---

With the introduction of the [Selectors API](http://www.w3.org/TR/selectors-api/) in HTML5, developers finally got a native means to selecting specific nodes without the need to traverse the DOM in tedious loops. The two new methods, `querySelector` and `querySelectorAll`, facilitate this functionality by querying the DOM via CSS selector strings. However, these new methods lack the performance of their closely related functions: `getElementById`, `getElementsByTagName`, and `getElementsByClassName`. This paired with the fact that effective CSS design should translate into an abundance of simple selectors that could be leveraged by one of these functions, and the unnecessary performance loss becomes much more apparent. For this article, I will be discussing the performance considerations for querying the DOM and interacting with the results, and offer a simple abstraction for an all-encompassing solution.

## Performance Considerations

To understand why `querySelectorAll` is slower than `getElementsByTagName` and `getElementsByClassName`, we have to dig into the internal mechanics that govern these functions. Consequently, the functions are actually handled very differently, returning different types of node lists. Both `getElementsByTagName` and `getElementsByClassName` return a `DynamicNodeList`, meaning it is _live_ and changes to the DOM will be automatically reflected in the collection. Conversely, `querySelectorAll` returns a `StaticNodeList` that serves as a snapshot of the DOM unaffected by changes. Static node lists must [collect the matched elements up-front](http://trac.webkit.org/browser/trunk/WebCore/dom/SelectorNodeList.cpp?rev=41093#L61) while live node lists do not. It is these pre-computations that directly attribute to the degradation in performance of `querySelectorAll`. 

It is not until a live node list is accessed, is the collection actually populated, such as using the [length property](http://trac.webkit.org/browser/trunk/WebCore/dom/DynamicNodeList.cpp?rev=41093#L57). In fact, this is the reason why iterating over both live and static node lists will yield the opposite performance results. Live node lists actually [prove slower](http://jsperf.com/getelementsbytagname-a-0-vs-queryselector-a/7) than static node lists because the DOM must be checked for changes for every element retrieved from the node list via the `item` method or bracket notation. However, iterations of live node lists will [perform better if first converted to an array](http://jsperf.com/nodelist-vs-array-iteration).

Even `querySelector` in comparison to `getElementById` results in a [considerable drop-off in performance](http://jsperf.com/getelementbyid-vs-queryselector). What's interesting is that the `querySelector` function does in fact map to `getElementById` internally if the selector string is a singular ID selector. So why the loss in performance? It turns out that even after `getElementById` is called, `querySelector` [continues to validate the returned element](http://trac.webkit.org/browser/trunk/WebCore/dom/Node.cpp?rev=41093#L1534) to ensure that it is in fact present in the DOM, a child element of the root element, and has the ID attribute. It would seem these additional hurdles are responsible for the performance hit.

## The Solution

The solution relies on leveraging the benefits of each function for each use case. This means improving performance for both DOM queries and iterations with the returned node list.

<div class="code-block">
  <pre class="prettyprint lang-javascript">
function query(selector, context) {
    context = context || document;
    // Redirect simple selectors to the more performant function
    if (/^(#?[\w-]+|\.[\w-.]+)$/.test(selector)) {
        switch (selector.charAt(0)) {
            case '#':
                // Handle ID-based selectors
                return [context.getElementById(selector.substr(1))];
            case '.':
                // Handle class-based selectors
                // Query by multiple classes by converting the selector 
                // string into single spaced class names
                var classes = selector.substr(1).replace(/\./g, ' ');
                return [].slice.call(context.getElementsByClassName(classes));
            default:
                // Handle tag-based selectors
                return [].slice.call(context.getElementsByTagName(selector));
        }
    }
    // Default to `querySelectorAll`
    return [].slice.call(context.querySelectorAll(selector));
}
</pre>
</div>

It works by first detecting a simple selector (anything that is solely an ID, class, or tag selector), then routing the selector string to the more performant function. Anything else is handled by `querySelectorAll`. The context of the query can be limited by providing a DOM element as an optional second argument. All of the following invocations would be interpreted as a simple selector and thus redirected to the faster method:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
query('#foo');
query('.foo');
query('.foo.bar');
query('div');
</pre>
</div>

The results are then converted into an array before returned. This not only makes the result easier to work with by making available all the methods of the `Array` prototype, but also makes it static for faster iterations.

Initial benchmarks indicate a sufficient improvement for DOM queries that are purely an [ID](http://jsperf.com/queryselectorall-vs-custom-query-method/4), [class](http://jsperf.com/queryselectorall-vs-custom-query-method/5), or [tag](http://jsperf.com/queryselectorall-vs-custom-query-method/6) selector. In comparison to live node lists, the same [performance gains are evident for iterations](http://jsperf.com/queryselectorall-vs-custom-query-method/7).

## Conclusion

Considering the usefulness of `querySelectorAll`, performance should be a concern, as any critical or frequently utilized code should be. However, despite its practicality, performance isn't the only concern. I tend to agree with John Resig, of jQuery fame, who points out some [flaws in the Selectors API specification](http://ejohn.org/blog/thoughts-on-queryselectorall/) when it was first created. Most notably is the element-rooted queries that match elements relative to the document and than filters them out based on whether or not the contextual element is an ancestor. This is completely absurd in my opinion as it directly impacts the reliability of `querySelectorAll` for complicated DOM trees and/or selector strings. This is another reason to rely primarily on simple selectors (ideally classes) to target the elements you intend to. Because of this, a simple abstraction such as the one above could prove highly beneficial to performance while also offering a more intuitive API for querying the DOM. For convenience, you can find the code on [GitHub](https://github.com/ryanmorr/query).