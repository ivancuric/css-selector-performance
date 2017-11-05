
# Optimizing CSS: ID selectors and other myths

In today’s typical scenario where the average website ships 500k of gzipped Javascript and 1.5MB of images, running on a midrange Android device via 3G with a 400ms RTT, CSS selector performance is the least of our problems.

Still, there’s something to be said about the topic, especially to weed out some of the myths and legends surrounding them. So let’s dive right in.

First, to get on the same page --- this article isn’t about the performance of CSS properties and values. What we’re covering today is the performance cost of the selectors themselves. I will be focusing on the Blink rendering engine, specifically Chrome 62.

The selectors can roughly be split in a few groups and (roughly) sorted from the least to most expensive.

1. ID `#classID`
2.- Class `.class`
- Tag `div`
- General and adjacents sibling `div ~ a`, `div + a`
- Child and descendant `div > a`, `div a`
- Universal `*`
- Attribute `[type="text"]`
- Pseudo-classes and elements, `a:first-of-type`, `a:hover`

Does this mean that you should only use IDs and classes? Well… Not really. Depends. First let’s cover how browsers interpret CSS selectors.
Browsers read CSS from right to left. The rightmost selector in a compound selector is know as the key selector. So for instance in `#id .class > ul a`, the key selector is `a`. The browser first matches all key selectors --- in this case it finds all elements on the page that match te `a` selector. It then finds all `ul` elements on the page and filters the `a`s to contain only those elements that are descendants of `ul`s, and so on until it reaches the leftmost selector. So here’s the first tip: the shorter the selector the better.

Ben Frain created a series of tests to measure selector performance back in 2014. The test consisted of an enormous DOM consisting of 1000 identical elements, and measuring the speed it took to parse various selectors, ranging from IDs to some seriously convoluted and long compound selectors. What he found was that the delta between the slowest and fastest selector was ~15ms.

However, that was back in 2014. Things have changed a lot since then, and memorizing rules is all but useless in the ever-changing browser landscape. Always remember to do your own tests, especially when performance is concerned.


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTk0MTA0MzYzMl19
-->