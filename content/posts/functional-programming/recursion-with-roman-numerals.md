---
categories:
- CS
date: "2017-07-17"
description: A procedure is recursive if it invokes itself. This article explains recursion in a simple way by examining functions that convert between Arabic and roman numerals.

slug: recursion-made-simple
tags:
- recursion
- javascript
title: Recursion Made Simple with Roman Numerals
draft: false
---

A procedure is recursive if it invokes itself. Thus:

```javascript
const toInfinityAndBeyond = num => toInfinityAndBeyond(num + 1);
```

Of course, `toInfinityAndBeyond` is only useful if, rather than seeking an answer, you want to blow your call stack. But you see the point: we find `toInfinityAndBeyond` in its own body. What possible use could this be?

Recursion is often well suited to express the logic of a problem. Let’s take a simple problem: the conversion of Roman to Arabic numerals. One solution is this one:

```javascript
function deromanize (str) {
	var	str = str.toUpperCase(),
		validator = /^M*(?:D?C{0,3}|C[MD])(?:L?X{0,3}|X[CL])(?:V?I{0,3}|I[XV])$/,
		token = /[MDLV]|C[MD]?|X[CL]?|I[XV]?/g,
		key = {M:1000,CM:900,D:500,CD:400,C:100,XC:90,L:50,XL:40,X:10,IX:9,V:5,IV:4,I:1},
		num = 0, m;
	if (!(str && validator.test(str)))
		return false;
	while (m = token.exec(str))
		num += key[m[0]];
	return num;
}
```

We might be interested in solving this problem differently for a number of reasons, perhaps to avoid the pitfalls that go along with complex regular expressions, or because we want to avoid mutating variables. Most relevant for our purposes is the key object. Note how it includes not only M:1000, but also entries like CM: 900 and IV: 4. This hard wires crucial logic rather than expressing it.

## A First Stab

Recursion gives us a more expressive way to approach the problem, one that allows us to use a single pure function that does not mutate variables. Here is a first stab:

```javascript
const nums = {I: 1, V: 5, X: 10, L: 50, C: 100, M: 1000};

const arabify = (romNum) => {
    if (romNum.length === 0) {
        return 0;
    } else if (nums[romNum[0]] < nums[romNum[1]]) {
        return (nums[romNum[1]] - nums[romNum[0]] + arabify(romNum.slice(2)));
    } else {
        return (nums[romNum[0]] + arabify(romNum.slice(1)));
    }
}
```

This has the benefit of expressing the logic of converting valid (or even some invalid!) Roman numerals to Arabic numerals, but it has some problems as well. Let’s consider the good before the bad.
The Good

In the first place, we are not using variables or complex regular expressions. More to the point, in our nums object, we have no need of shortcuts like IX: 9 or CM: 900 — our program handles this for us. How?

The function `arabify` takes a string, which we are assuming to be valid Roman numerals. If that string is empty, it simply returns 0. An empty string is our base case, the point at which `arabify` stops calling itself. Without a base case, the `arabify` calls will just keep piling up on the stack until there is a stack overflow.

If the string passed to `arabify` is not empty, we must determine the relation of the first two characters. If the first character is less than the second character — `nums[romNum[0]] < nums[romNum[1]` — we know that the former should be subtracted from the latter. That is, if X represents 10 and C a 100, then XC is 90.

Why in the case where the first two characters require the first be subtracted from the second do we need the call again to `arabify`? Because we could have a case like XCII, where we not only need to subtract the first item from the second, but the result must be added to the remaining numerals.

If the first numeral is not less than the second numeral, the solution is straightforward: add the first numeral to the value of the rest. We can simply call `arabify` on the remainder of the string from Roman numerals to find that value.

## The Bad

One downside to `arabify` is that it’s not the most readable. Wouldn’t it be nice if instead of having to parse things like `nums[romNum[0]] < nums[romNum[1]`, we could have `nums[fst] < nums[snd]` instead?

The second problem is unlikely to arise in this particular context, but pointing it out is useful. The calls to `arabify` are going to keep piling on the call stack until the base case is reached — an empty string, then each call can be evaluated and the final answer returned. This means that `arabify` will consume memory roughly proportionally with the size of the string passed in as an argument.

Fortunately, the ES6 spec now includes tail call optimization. For recursive calls in the tail position, the function calls need no longer pile up on the stack. Let’s take a second stab at `arabify` that makes this clearer.

## Tail Call Optimization

arabify2 uses tail calls and is a bit easier to read.

```javascript
const nums = {I: 1, V: 5, X: 10, L: 50, C: 100, M: 1000};

const arabify2 = (romNum, sum=0) => {
    const [fst, snd, rest] = [romNum[0], romNum[1], romNum.slice(2)];
    if (!snd) {return nums[fst] ? nums[fst] + sum : sum;}
    else if (nums[snd] > nums[fst]) {
        return arabify2(rest, nums[snd] - nums[fst] + sum);
    } else {return arabify2(snd + rest, nums[fst] + sum);}
}
```

On line four, we use ES6 destructuring to so that we can say fst rather than romNum[0], snd rather than romNum[0], and rest rather than romNum.slice(2) or romNum.slice(1).

Note the difference between the recursive calls to `arabify2` in comparison to `arabify`: we don’t make a call to `aribify2`and then do something else with it. `aribify2` uses the parameter sum to keep track of all the information it needs. There is no reason to ‘remember’ the previous aribify2 calls: the last call returns the result we are looking for.

