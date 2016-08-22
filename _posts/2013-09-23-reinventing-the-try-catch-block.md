---
layout: post
title: Reinventing the Try/Catch Block
date: 2013-09-23
project: https://github.com/ryanmorr/try-catch
tags:
  - JavaScript
---

The try/catch block is a unique construct, both in how it works and what it is capable of. Fundamentally, it is able to isolate one or more statements to capture and suppress any runtime errors that may be encountered as a result of execution. It is such a powerful construct that in a perfect world you would want to wrap everything in a try/catch block to provide simple and effective error trapping. However, due to concerns in performance critical situations, employing the construct is often frowned upon. But what if I told you there was a means of emulating the try/catch block without the concern for performance? This article will be exploring just such a method.

## Performance Implications of the Try/Catch Block

What makes this construct unique is in the manner in which the catch block augments the scope chain. Rather than creating a new execution context and pushing it to the top of the execution stack, the catch block will actually create a new variable object and place it ahead of the activation object in the scope chain of the current execution context. This creates what is known as a _dynamic scope_, similar to the effect of the `with` statement, which lends to its bad reputation as well. As a result, the error object passed to the catch block does not exist outside of it, even within the same scope. It is created at the start of the catch clause and destroyed at the end of it. This type of manipulation of the scope chain is the primary contributor to the performance hit.

At this point you may be thinking that as long as an error is not raised than performance should not be affected, a fair assumption, but you'd be wrong. Some JavaScript engines, such as V8 (Chrome) do not optimize functions that make use of a try/catch block as the optimizing compiler will skip it when encountered. No matter what context you use a try/catch block in, there will always be an inherent performance hit, quite possibly a substantial one.

These limitations are well documented, for instance, look at the following test cases: <http://jsperf.com/try-catch-block-performance-comparison> and <http://jsperf.com/try-catch-block-loop-performance-comparison>. The former confirms that not only is there up to a 90% loss in performance when no error even occurs, but the declination is significantly greater when an error is raised and control enters the catch block. The latter test case proves that the loss is compounded in loops, where most performance intensive operations typically occur.

## Global Error Handling Using `window.onerror`

To find a suitable alternative, we require a reliable means of error notification. For this, there is really only one viable source in the browser we can turn to for error trapping, that being `window.onerror`. The question is, can we leverage the event as a means of closely mimicking the functionality of a try/catch block? The answer is yes&#8230; for the most part. The event is capable of detecting runtime errors including any errors that you explicitly throw yourself. We can also imitate the error suppression of a catch block by returning false from `window.onerror`, while returning true allows the error to propagate to the browser.

For a long time, despite the cross-browser support and somewhat consistent behavior, the `window.onerror` event was non-standard. However, this has changed as the [event has been recently added to the spec](https://html.spec.whatwg.org/multipage/webappapis.html#errorevent). This bodes well for the future and a more predictable API to depend on as browsers slowly adopt the convention defined by the specification.

## The Solution

In its simplest form, the solution involves temporarily reassigning the `window.onerror` event handler before invoking a “try” function. This allows us to reliably resolve the source of an error when one is encountered. At this point, the associated “catch” handler is invoked, passing the error object as the only argument:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
function tryCatch(tryFn, catchFn) {
    // Cache current `onerror` callback
    var prev = window.onerror;
    // Define a new callback for global `onerror` events
    window.onerror = function(msg, file, line, col, err) {
        // Restore original `onerror` callback
        window.onerror = prev;
        // If the error object was not provided, than create it
        var error = err || new Error();
        // uniform error properties across browsers
        error.message = msg;
        error.fileName = file;
        error.lineNumber = line;
        error.columnNumber = col;
        // Invoke the "catch" function passing the error object
        var suppress = catchFn(error);
        // If the callback returned false then allow the error to 
        // propagate to the browser, otherwise suppress it like a native 
        // try/catch block would
        return suppress === false ? false : true;
    };
    // Invoke the "try" function
    tryFn();
    // Restore original `onerror` callback
    window.onerror = prev;
}
</pre>
</div>

The function, appropriately named `tryCatch`, allows us to wrap error-prone or sensitive code in a similar fashion as a try/catch block. The syntax is similar as well, the exception being the use of functions instead of block statements, such as the following:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
tryCatch(function(){
    // Try something    
}, function(error){
    // Handle the error
});
</pre>
</div>

The error object passed to the callback has the added benefit of retaining the file name, line number, and column number of the location of the error, something that is widely inconsistent amongst browsers when employing a try/catch block. Additionally, if the error object is provided to the `window.onerror` callback as a parameter, the name, stack trace, and other properties will be available to further improve our ability to debug an error.

## Performance Comparison

When we measure the performance of this custom solution against a native try/catch block, we get some interesting results. Based on some preliminary tests, I found performance comparisons between the browsers fairly inconsistent. Firefox, IE, and Opera all showed improved performance using the `tryCatch` function as opposed to a native try/catch block, while the results were opposite for Chrome and Safari. However, when we avoid needlessly creating anonymous functions for every invocation of the `tryCatch` function and instead use predefined functions, performance reaches near native speeds in most browsers. Check out the results and try it yourself at: <http://jsperf.com/native-try-catch-vs-custom-try-catch/7>. The real advantages come inside loops, in this case performance increased dramatically in most browsers: <http://jsperf.com/native-try-catch-vs-custom-trycatch-loop>.

## Drawbacks

Despite some of the advantages to this approach, it is not without its caveats. One such disadvantage is when the “catch” handler is invoked in the case of an error and another error occurs within the handler, then both errors will propagate to the browser. This happens because control still hasn't left the `window.onerror` event handler from the initial error, raising another error will stop execution and resort to the default behaviour. This makes nested `tryCatch` calls infeasible.

Another drawback inherent to `window.onerror` is that once an error is encountered, the browser will stop execution following the invocation of the event handler. Any code following the `tryCatch` call will be skipped as the interpreter will instead proceed to the next script block (`<script></script>`) if it exists. This is unavoidable, only a catch block is capable of truly suppressing an error without halting execution. Of course, if no further execution is required than this drawback is irrelevant.

One concern that should be cleared up by the new specification are the question marks surrounding `window.onerror`. For example, it’s not even clear what errors trigger the event and which don’t. The official Mozilla documentation supports this: 

> Note that some/many `error` events do not trigger `window.onerror`, you have to listen for them specifically.

Unfortunately, Mozilla doesn't get any more specific in terms of which errors are actually caught by `window.onerror`. According to the [Internet Explorer documentation](http://msdn.microsoft.com/en-us/library/ie/cc197053%28v=vs.85%29.aspx), the following errors trigger the event:

>   * Run-time script error, such as an invalid object reference or security violation.
>   * Error while downloading an object, such as an image.
>   * Windows Internet Explorer 9. An error occurs while fetching media data.

This seems to be consistent with all browsers, and testing has yet to reveal an exception. That's not bad, but the lack of a clear definition is a little troubling, therefore the question still remains.

## Conclusion

The performance benefits of this function are debatable. However, one genuine benefit to this method is in the numerous opportunities for customization. The ability to tailor the `tryCatch` function for different use cases and meet various requirements you otherwise couldn't achieve with a try/catch block.

You could look at it as a suitable median between a native try/catch block and a traditional `window.onerror` event handler. On one hand, you can mimic a try/catch block but gain the opportunity to enhance functionality while also proving more beneficial in performance critical situations in some circumstances. On the other hand, it can narrow the scope of a `window.onerror` event to specific fragments of code for more effective debugging. Kind of a best of both worlds solution.

Now let me be clear, I am absolutely NOT advocating that you replace all your try/catch blocks using this method. This is still very much in the experimental stage, relying on this method in any type of production environment would be premature. I would, however, encourage you to [fork the code on Github](https://github.com/ryanmorr/try-catch). Maybe you will find some practical use cases in a real world application.