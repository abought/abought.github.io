---
layout: post
title:  "Yes, no, and maybe (except)"
date:   2020-10-17 17:06:25 -0400
categories: javascript learning
---

When learning to use a new language or tool, I tend to organize the information into two pieces: deep insights, and a knowledge of quirks. But while this is useful for myself, I am always skeptical of those who think this is a good way to teach (or evaluate) someone else: it often leads to giving "the weird stuff" too much weight.

Consider a brief JavaScript trivia question:

*Do the keys in a JavaScript Object preserve insertion order?*

* A. Yes
* B. No
* C. Maybe


{% highlight javascript %}
> const myobject = {c: 3, a: 1, b: 2};
> console.log(myobject);
{% endhighlight %}


There are three ways that one could answer a question like this:
1. *Empirical*: Try it out in a browser,  and see what happens. Oftentimes, browsers will converge on a similar behavior, even where the specification treats something as an implementation detail
2. *Theoretical*: Consult the language specification for what is (or eventually will be) the authoritative answer
3. *Paranoid*: Consider edge cases. For mission critical code, recognize that evolving specifications may not be implemented everywhere (or at all)

## Empirical
Empirically, the answer is a weak "yes"; many browsers have [preserved](https://bugs.chromium.org/p/v8/issues/detail?id=164) the insertion order of keys for years. If you construct an object with keys in insertion order, then (usually), the keys are iterated in the same order.

{% highlight javascript %}
> Object.keys(myobject);
["c", "a", "b"]
{% endhighlight %}

It’s tempting to stop there. For many cases, a quick experiment like this can be "good enough".

## Theoretical
Such questions can often be answered definitively by consulting the official specification for how things should work. But JavaScript (and browsers) are challenging in part because the language [changes over time](https://stackoverflow.com/a/58444013): according to various versions of the ECMA262 (ECMAScript) language specification, the answer is somewhere on the continuum from no ("implementation detail") to [yes](https://github.com/tc39/ecma262/pull/1791) (“see pseudocode”). Since JavaScript runs in the browser, the runtime is not under the control of the programmer; just about every possible version of the rules will apply to some subset of your users. This leads to a conservative and defensive style in which “new” features take many years to become reliably useful. Transpilers like Babel can help, but they are most useful with syntax. At some level, internal language details are beyond the power of static analysis tools to fake.

## The weird stuff
And then, there’s the paranoid answer: it depends.  Let’s take the example above, and swap keys and values. In Javascript, all object keys are actually strings, so we will wrap our numeric keys in quotation marks for conceptual clarity.
{% highlight javascript %}
> const myobject = {'3': 'c', '1': 'a', '2': 'b'};
{1: "a", 2: "b", 3: "c"}
> Object.keys(myobject)
["1", "2", "3"]
{% endhighlight %}

As of ECMAScript 2015, it is an official part of the specification that “integer index” strings are treated differently than other strings. (see [summary](https://stackoverflow.com/a/54670669) and  [official spec details](https://www.ecma-international.org/ecma-262/9.0/index.html#sec-ordinaryownpropertykeys)). Consider: in Javascript, Arrays are a special type of object- `list[0]` can be regarded as syntactic sugar for “get me the value at key ‘0’”. Since object keys are actually strings, there’s some type coercion magic involved here too. Using a string that *looks* like a number will trigger array-like ordering behavior. 

This isn’t just a fun parlor trick; it can affect the behavior of running code.  In our LocusZoom.js rendering code, a custom track was positioning items according to unique values from a specific text file column, tracked in a hash. This worked out relatively well, until a new file came in that used numeric instead of human-friendly labels. Suddenly, the display order of fields didn’t work in the way we expected: a difference in user input strings triggered a different language behavior for the same data structure!

### Conclusions
For the sake of the original trivia question we started with: depending on your problem to be solved, there are three possible answers, and *none* of them are accurate. As of this writing, the most *accurate* answer to our question is “Maybe (except)”.  

But programming isn't about answering a quiz: we are continuously evolving our tools and work habits in the pursuit of *what works*. If you are writing real code that requires tracking insertion order: consider the [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) data type when you need to worry about key order at all.

{% highlight javascript %}
> const myobject = new Map([
    ['3', 'c'],
    ['1', 'a'], 
    ['2', 'b'],
]);
{"3" => "c", "1" => "a", "2" => "b"}
> myobject2.keys()
MapIterator {"3", "1", "2"}
{% endhighlight %}

Particularly in a fast-evolving language, there are many traps that are difficult to remember. It’s tempting to treat programming knowledge as a trivia contest, but it’s better to define coding standards that guide new teammates around these pitfalls altogether. Too many style discussions and rules focus on whitespace control (tabs! spaces!) instead. If you are a senior developer: how can you turn the discussion away from trivia, and towards laying the groundwork for your team to succeed?
