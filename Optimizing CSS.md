
# Optimizing CSS: ID selectors and other myths

In today’s typical scenario where the average website ships 500k of gzipped Javascript and 1.5MB of images, running on a midrange Android device via 3G with a 400ms RTT, CSS selector performance is the least of our problems.

Still, there’s something to be said about the topic, especially to weed out some of the myths and legends surrounding them. So let’s dive right in.

## The basics of css parsing

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

Browsers read CSS from right to left. The rightmost selector in a compound selector is know as the key selector. So for instance in `#id .class > ul a`, the key selector is `a`. The browser first matches all key selectors --- in this case it finds all elements on the page that match te `a` selector. It then finds all `ul` elements on the page and filters the `a`s to contain only those elements that are descendants of `ul`s, and so on until it reaches the leftmost selector.

So here’s the first tip: the shorter the selector the better, and make sure that if possible, you keep the key selector a class or ID.

## Measuring the performance

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

What we can see from this test is that it's not really worth it to worry over CSS selector performance - just don't overdo it with pseudoselectors and really long selectors. We can also see how Blink improved in the last 2 years. Instead of the stated ~500x slowdown for a "quantity selector" (`.box:nth-last-child(n+6) ~ div`) compared to an "insanity selector" (`.box:not(:empty):last-of-type .title`), we only see a ~1.5x slowdown. That's an amazing improvement, and we can only expect browsers to get better, making CSS selector performance even less impac

You _should_ however stick to using classes whenever possible, and adopt some sort of namespacing convention like BEM, SMACSS, OOCSS since it will not only help your website's performance but vastly help with code maintainability. Overqualified compound selectors, especially when used with tag and universal selectors, eg `.header nav ul > li a > .inner` are extremely brittle and a source of many unforseen errors. They are also a nightmare to maintain, especially if you inherit it from someone else.

## Quality over quantity

A bigger problem of simply having expensive selectors is having _a lot_ of them. This is know as "style bloat", and you've probably seen the problem a lot. Typical examples are sites which import entire CSS frameworks like Bootstrap or Foundation, while using less than 10% of the transfered CSS. Another example are old, never refactored projects whose CSS has devolved into, as I like to call them, "Chronological Style Sheets" - CSS with a ton of appended classes to the end of the file as the project changed and grew, now looking more like an overgrown garden full of weeds.

Not only does a large CSS file take longer to transfer, (and network is the _biggest_ bottleneck in website performance), they also take longer to parse. As well as constructing the DOM from your HTML, the browser needs to construct a CSSOM (CSS Object Model) to compare it with the DOM and match the selectors.

Today's browsers

If you want to go a bit more in depth about how the browsers parse CSS, [check out Nicole Sullivan's post on Webkit](https://calendar.perfplanet.com/2011/css-selector-performance-has-changed-for-the-better/), [Ilya Grigorik's article on how Blink does it](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/constructing-the-object-model) , or [Lin Clark's article on Mozilla's new Stylo CSS engine](https://hacks.mozilla.org/2017/08/inside-a-super-fast-css-engine-quantum-css-aka-stylo/). 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3MDM1MzI0MDFdfQ==
-->