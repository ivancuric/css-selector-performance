
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

I went to do my own tests, and for that I used Paul Lewis' test mentioned in [Paul Irish's comment](https://alistapart.com/comments/quantity-queries-for-css#338752) expressing concern on the useful, yet convoluted "[quantity selectors](https://alistapart.com/article/quantity-queries-for-css)". :

> These selectors are among the slowest possible. ~500 slower than something wild like “div.box:not(:empty):last-of-type .title”. Test page http://jsbin.com/gozula/1/quiet

The test was bumped up a bit, to 50000 elements, and you can [test it out yourself](https://codepen.io/ivancuric/pen/ZaWxqV). I did an average of 10 runs on my 2014 MacBook Pro, and what I got was the following:
| Selector | Query Time (ms) |
|--|--|
| `div` | 4.8740234375 |
| `.box` | 3.625 |
| `.box > .title` | 4.458740234375 |
| `.box .title` | 4.51611328125 |
| `.box ~ .box` | 4.708251953125 |
| `.box + .box` | 4.6611328125 |
| `.box:last-of-type` | 3.94482421875 |
| `.box:nth-of-type(2n - 1)` | 16.84912109375 |
| `.box:not(:last-of-type)` | 5.894775390625 |
| `.box:not(:empty):last-of-type .title` | 8.020263671875 |
| `.box:nth-last-child(n+6) ~ div` | 20.87109375 |

The results will of course vary depending if you use `querySelector` or `querySelectorAll`, and the number of elements, but `querySelectorAll` comes closer to the real use case of CSS. 

Even  in such an extreme case, with 50000 elements to match, and using some really insane selectors like the last one, we find that the slowest one is ~20ms, while the fastest is the simple class at ~3.5ms. Not really that much of a difference. In a realistic, more "tame" DOM, with around 1000 - 5000 nodes, you can expect those results to drop by a factor of 10, bringing them to sub-millisecond parsing speeds.

What we can see from this test is that it's not really worth it to worry over CSS selector performance - just don't overdo it with pseudoselectors and really long selectors. You should however stick to using classes whenever possible, and adopt some sort of namespac
<!--stackedit_data:
eyJoaXN0b3J5IjpbODAyNDYyMTUzXX0=
-->