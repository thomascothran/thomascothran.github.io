---
categories:
- CS
date: "2017-07-17"
description: The polymath Blaise Pascal envisaged a triangle built of numbers. Pascal’s triangle — as it is usually called, despite the fact that its discovery predated Pascal by centuries — has the interesting property that each number is the sum of the two numbers directly above it.
slug: haskells-triagle-introduction-to-recursion
tags:
- recursion
- haskell
title: Haskell's Triangle
draft: false
---

The polymath Blaise Pascal envisaged a triangle built of numbers. Pascal’s triangle — as it is usually called, despite the fact that its discovery predated Pascal by centuries — has the interesting property that each number is the sum of the two numbers directly above it.

In this post we will use Pascal’s triangle to demonstrate how recursion (i.e., a procedure that invokes itself in its definition) can be used to make complex problems easily soluble, using examples written in both Haskell and JavaScript.
Wikipedia

Pascal’s triangle is constructed such that the first row has one number: 1. Each additional row adds one additional number. The third row, therefore, has three numbers; the 1,542nd row has 1,542 numbers. Each number is the sum of the two numbers above it, except for the 1 at the pinnacle.

If we want to solve a problem recursively, we start with what is easy. What is obvious about Pascal’s triangle? Well, to begin with, we have simply posited that the number in row 1 is 1. (This is the only number in the triangle that is not the sum of the numbers above it.) We also see that first and last column on each row will be a 1. The reason for this is that each of these is the sum of 1 and nothing (i.e, 0).

Let’s say we need a function — let’s call it pt — that takes two integers, one representing a row and one the column. pt should return the number at that position. For instance, pt 1 1 should return 1, and pt 3 2 should return 3. (Note that we are not using zero based numbering for our rows and columns).

Our type signature will be `pt :: (Integer a) => a -> a -> a`. That is, is pt a function that takes two integers and returns an integer.
Start with the easy answers

When coming up with a recursive solution, we start with the easy answers. What’s the easiest case here? Well, the number at row 1 column 1 is 1!

```haskell
pt :: Integer -> Integer -> Integer
pt 1 1 = 1// JavaScript
function pt(row, col) {
  if (a === 1 || b === 1) {
    return 1;
  }
}
```

That was easy. What else can we say for sure? For one thing, there is nothing at a column less than 1. Nothing = zero. So if we ask pt for a position where the column is less than 1, pt should return 0.

```haskell
pt :: Integer -> Integer -> Integer
pt row col
  | row == 1 && col == 1 = 1
  | col < 1 = 0// JavaScript
function pt(row, col) {
  if (row === 1 && col === 1) {
    return 1;
  } else if (col < 1) {
    return 0;
}
```

Come to think of it, if the column position is less than 1, that is just a special case where the number is outside Pascal’s triangle (in this case, to the left). What if the column position is outside the triangle to the right side? It would also be zero. We know that the total number of columns in a row is equal to the row number. Thus:

```haskell
pt :: Integer -> Integer -> Integer
pt row col
  | row == 1 && col == 1 = 1
  | col < 1 || col > row = 0// JavaScript
function pt(row, col) {
  if (row === 1 && col === 1) {
    return 1;
  } else if (col < 1 || col > row) {
    return 0;
  }
}
```

After the easy questions, then what?

The nice thing about recursion is that we can just state what we know, and the computer will just figure out what we don’t. We know that any column outside the triangle will equal zero, and we know the triangle’s pinnacle is one.

We don’t need to manually figure out the numbers for any other position; we can just ask the computer to figure it out for us. All we need to do is describe to the computer what we need it to find. We can treat the program as magic, or the computer as an oracle.

```haskell
pt :: Integer -> Integer -> Integer
pt row col
  | row == 1 && col == 1 = 1
  | col < 1 || col > row = 0
  | otherwise = (pt (row - 1) (col - 1)) +
                (pt (row - 1) (col))// JavaScript
function pt(row, col) {
  if (row === 1 && col === 1) {
    return 1;
  } else if (col < 1 || col > row) {
    return 0;
  } else {
    return (pt (row - 1, col - 1)) +
           (pt (row - 1, col));
  }
}
```

Magic!

And there we have it! `pt 1 1` returns 1, `pt 3 2` returns two, and `pt 17 5` returns 1820.

But it’s not really magic. There are a few things to keep in mind when constructing recursive functions, most importantly the base case imperative. Our base cases return answers, rather than recursively invoking the function. If we don’t hit the base cases eventually, the program will never terminate. The fact that the recursive call here decrements row each time means that we will eventually hit the first row.

The other thing to keep in mind is that this sort of recursive call is a form of tree recursion. The larger the numbers, the more resources the procedure will demand. The rate of growth here is exponential. My computer quickly gave me the answer to `pt 17 5`; I’m still waiting on the answer to `pt 71 5`.
