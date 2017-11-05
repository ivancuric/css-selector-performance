
# Optimizing CSS: ID selectors and other myths

In today’s typical scenario where the average website ships 500k of gzipped Javascript and 1.5MB of images, running on a midrange Android device via 3G with a 400ms RTT, CSS selector performance is the least of our problems.

Still, there’s something to be said about the topic, especially to weed out some of the myths and legends surrounding them. So let’s dive right in.

First, to get on the same page --- this article isn’t about the performance of CSS properties and values. What we’re covering today is the performance cost of the selectors themselves. I will be focusing on the Blink rendering engine, specifically Chrome 62.

The selectors can be split in a few groups and (roughly) sorted from the least to most expensive.

1. ID `#classID`
2. Class `.class`
3. Tag `div`
4. General and adjacents sibling `div ~ a`, `div + a`
5. Child and descendant `div > a`, `div a`
6. Universal `*`
7. Attribute `[type="text"]`
8. Pseudo-classes and elements, `a:first-of-type`, `a:hover`

Does this mean that you should only use IDs and classes? Well… Not really. Depends. First let’s cover how browsers interpret CSS selectors.

Browsers read CSS from right to left. The rightmost selector in a compound selector is know as the key selector. So for instance in `#id .class > ul a`, the key selector is `a`. The browser first matches all key selectors --- in this case it finds all elements on the page that match te `a` selector. It then finds all `ul` elements on the page and filters the `a`s to contain only those elements that are descendants of `ul`s, and so on until it reaches the leftmost selector. So here’s the first tip: the shorter the selector the better, and make sure that if possible, you keep the key selector a class or ID.

Ben Frain created [a series of tests](https://benfrain.com/css-performance-revisited-selectors-bloat-expensive-styles/) to measure selector performance back in 2014. The test consisted of an enormous DOM consisting of 1000 identical elements, and measuring the speed it took to parse various selectors, ranging from IDs to some seriously complicated and long compound selectors. What he found was that the delta between the slowest and fastest selector was ~15ms.

However, that was back in 2014. Things have changed a lot since then, and memorizing rules is all but useless in the ever-changing browser landscape. Always remember to do your own tests, especially when performance is concerned.

I went to do my own tests, and for that I used Paul Lewis' test mentioned in [Paul Irish's comment](https://alistapart.com/comments/quantity-queries-for-css#338752) expressing concern on the convoluted "[quantity selectors](https://alistapart.com/article/quantity-queries-for-css)". 

The test was bumped up a bit, to 50000 elements, and you can [check it out yourself](https://codepen.io/ivancuric/pen/ZaWxqV). I did an average of 10 runs, and what I got was the following:
|  |  |
|--|--|
|div | 30.5400390625ms |
|.box | 4.708984375ms |
|.box > .title | 5.144775390625ms |
|.box .title | 4.68212890625ms |
|.box ~ .box | 4.846923828125ms |
|.box + .box | 4.919189453125ms |
|.box:last-of-type | 3.85400390625ms |
|.box:nth-of-type(2n - 1) | 18.135009765625ms |
|.box:not(:last-of-type) | 5.64208984375ms |
|.box:not(:first-of-type) | 5.126953125ms |

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTgyNDkwNzc3NV19
-->