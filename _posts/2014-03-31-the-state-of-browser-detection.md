---
layout: post
title: The State of Browser Detection
date: 2014-03-31
categories:
  - Articles
tags:
  - Browser Detection
  - Bugs
  - Feature Testing
  - Graceful Degradation
  - JavaScript
  - Progressive Enhancement
  - Tips
---

Despite the progress client-side scripting has made in the last decade or so, it seems some bad practices are poised to never die. With the medium transitioning into a more mobile-centric world in recent years, an influx in bugs has many turning back to browser detection for a solution. History seems to repeat itself in this regard. It has long been my contention that browser detection should not and can not be relied upon. However, the validity of the use case has become difficult to dismiss, and the lack of an alternative leaves me with no other options and more than a little conflicted. For this article, I will be discussing the various facets of browser detection and offer some recommendations to maximize reliability.

## A Brief Overview

Browser detection is the practice of attempting to identify the web browser a visitor is using. It&#8217;s purpose is to either impose or restrict execution of certain fragments of code to a particular browser in an effort to circumvent incompatibilities or leverage features associated with that specific browser. This exercise is not only restricted to the browser vendor, but also the version, operating system, rendering engine, and device. 

There are two predominant approaches to browser detection. The most common method is by way of _user-agent sniffing_. This describes the parsing of the user-agent string contained in the `navigator.userAgent` property to extract the relevant browser data.

The other approach is in the form of _browser inference_, which is the identification of a particular browser or version based on the presence or absence of one or more unique objects. A common example of this would be inferring Internet Explorer based on the existence of the `ActiveXObject` object.

## Early History of Browser Detection

Early on as JavaScript began to gain traction, it found itself torn between competing browsers. As a result, implementations featured notable inconsistencies, various bugs and quirks, and fickle web standards compliance. Many developers found themselves resorting to browser detection to help abstract away the incompatibilities. At the time, it was viewed as a justified solution to obtain cross-browser compatibility. However, at some point along the way, the practice became abused, quickly becoming common routine rather than a last resort, finding its way into scripts where simple object detection would suffice. JavaScript libraries would unnecessarily incorporate browser detection, adding a certain air of credibility to it. This all hindered the Web from progressing as fast as it could have, contributing in part to the long and overdue lifespan of IE6. 

## Assumptions can be Dangerous

Before I concede to browser detection, I would first like to remind or enlighten those less familiar as to why it is untrustworthy. The fundamental problem with browser detection is in its presumptuous nature. It&#8217;s a static identifier operating in an environment that is constantly evolving. Both the browser detect itself and the features/bugs associated with each browser are only as reliable as your knowledge of the browser environments you currently support at the time of implementation. This essentially boils down to nothing more than assumptions about every browser of every user that visits a page, and assuming the relationships between features/bugs and the associated browsers are correct affiliations. As the technology continues to evolve and new browsers and browser versions released, applications depending on browser detection will become fragile implementations at the mercy of the progression. 

Additionally, user-agent sniffing poses an entirely new challenge in the form of _browser spoofing_. This is the practice of one browser attempting to [imitate another by polluting the user-agent string](http://webaim.org/blog/user-agent-string-history/) with keywords that hint at the existence of different browser vendors, rendering engines, and operating systems. Add to this the fact that the user-agent string can be manipulated, with or without the user&#8217;s knowledge, and it becomes increasingly clear how unreliable this technique truly is. In essence, it amounts to nothing more than a best guess.

## Feature Testing is the Answer

There is no question that _feature testing_ is the preferred means of detecting bugs and supported functionality of the browser, that debate was settled a long time ago. It is the logical concept of testing for the existence, and in some respects, the functionality of a feature before utilizing it in some capacity. It relies on what it knows about the browser environment during runtime rather than the assumed associations at the time of implementation. jQuery came to the same conclusion, [dropping browser detection](http://blog.jquery.com/2009/01/14/jquery-1-3-released/) from the core library in favor of feature testing, finally acknowledging it as the more reliable approach. Given the possibility to form a meaningful test case, feature testing is the obvious answer. Of course this bears the question, what if there is no applicable feature test available? What if the feature/bug is undetectable?

## Undetectables

This is an instance of a bug beyond the scope of JavaScript, meaning the formation of any competent test case would be infeasible. The bug is simply undetectable, leaving only browser detection to single out the affected browsers. This forms the basis for the validity of the use case for browser detection, which speaks to its prevalence in scripts to this day.

This isn&#8217;t exactly a new problem, however, it does seem to be a growing problem since the emergence of mobile browsers. There are numerous [undetectable bugs and quirks](https://github.com/Modernizr/Modernizr/wiki/Undetectables) to speak of. Perhaps the most notable due to its sheer usefulness in modern day web applications is fixed positioning. Its issues are [well](http://remysharp.com/2012/05/24/issues-with-position-fixed-scrolling-on-ios/) [documented](http://bradfrostweb.com/blog/mobile/fixed-position/) on both iOS and Android. The [browser detect required for determining fixed positioning support](https://github.com/jquery/jquery-mobile/blob/f6d4e37fb22f9d5c7393b4ee98eb9ca1c836ebcf/js/support.js#L143-L173) is complicated to say the least, which speaks to the extent of the problem. It seems to be the nature of mobile development at this time, much like it was when desktop browsers were still in their infancy. At least the rationale for browser detection is of greater merit today than it was a decade ago.

## Strengthening Browser Detection

First and foremost, browser detection should always be a last resort. However, if it is your only option, you can employ the technique more safely, provided you take the proper steps to help mitigate the inherent risks in reliability.

One such step would be to employ a more direct form of browser inference. Rather than rely on an unrelated object, why not use a more relevant object to infer the browser, one that implies details about a specific user-agent. For example, using vendor prefixes to infer the rendering engine or other proprietary objects whose names hint at the vendor they belong to:

<div class="code-block">
  <pre class="prettyprint lang-javascript">
if('chrome' in window){
    // detect Chrome
}
if('opera' in window && ({}).toString.call(window.opera) === '[object Opera]'){
    // detect Opera 14-
}
if('PalmSystem' in window){
    // detect webOS
}
if('blackberry' in window){
    // detect blackberry
}
if('MozAppearance' in document.documentElement.style){
    // detect Mozilla
}
if('WebkitAppearance' in document.documentElement.style){
    // detect WebKit
}
</pre>
</div>

This serves as a much more straightforward means of identifying the browser. It is still an assumption, but it&#8217;s one I&#8217;m willing to depend on. I don&#8217;t necessarily see this as a replacement for user-agent sniffing, however, if utilized conjointly, it can help to fortify the detection process and reduce, if not eliminate, false positives and false negatives. 

Additionally, [conditional comments](http://msdn.microsoft.com/en-us/library/ms537512.ASPX) have long proved to be a reliable means of detecting Internet Explorer and different versions of IE. There is also the [JavaScript equivalent](https://gist.github.com/scottjehl/358029) available, all but eliminating the requirement to sniff out old IE via user-agent sniffing or browser inference. Unfortunately, [IE10 and later have dropped support for conditional comments](http://msdn.microsoft.com/en-us/library/ie/hh801214(v=vs.85).aspx), which means you may be forced to turn back to the aforementioned and less desirable methods. 

## Optimal Implementation

Aside from the code that makes up the browser detect itself, how you implement it can be equally critical to reliability. Ideally, features should be employed in a fashion that does not inhibit the user&#8217;s experience in the event that the depending browser detect fails. This begins by adopting one of two fail-safe strategies; _progressive enhancement_ or _graceful degradation_. Both concepts help to make a web site accessible to any browser, while providing improved aesthetics and/or usability for more capable browsers. 

The difference between the two is in the approach. Progressive enhancement entails development in layers, using a bottom-up approach. Starting with a cross-browser base layer, features are added in successive layers to enhance the web site. Graceful degradation uses a top-down approach in which features are implemented with a fallback solution for nonsupporting browsers. Both strategies help to ensure a functional and legible site even under the worst case scenario, establishing a high level of consistency.

Be precise and target the specific version of a specific browser. Narrow the scope of your detection to the exact browsers you know are comprised of the feature/bug you are attempting to account for. This helps avert a wide reaching browser detect that is likely to break in the future. Also be sure to avoid browser detects based on assumptions about non-existent future versions of a browser. Instead, work off of what you know at the time and don&#8217;t try to guess the future. If the current version of a particular browser has a bug, do not assume that bug will be present in the next version. This serves to future proof your code. 

## Conclusion

To put it bluntly, I suspect browser detection is here to stay. Bugs are inevitable, it is the nature of software. Unexpected defects and quirks will always necessitate the demand to identify specific browsers when feature testing is not applicable. The use case is valid. What needs to change is the method in which the browser is identified. At present, developers are forced to contend with the overly convoluted user-agent string that insists on misrepresenting itself. This makes for an unreliable technique to hinge critical functionality on. Regardless of how it is employed, if the browser detect is, in of itself, untrustworthy, than it&#8217;s a flawed approach. Yet, without a comparable alternative, developers have no where else to turn. This leaves two options; avoid the broken features completely or use browser detection. Not exactly an ideal position anyway you approach it. We need something better!