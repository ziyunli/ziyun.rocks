---
title: "Strategic vs Tactical Programming"
date: 2021-06-18
tags: [software-engineering]
categories: [notes]
---

I am reading [A Philosophy of Software Design by John Ousterhout](https://www.goodreads.com/en/book/show/39996759), and the chapter 3 about strategic vs. tactical programming resonates a lot with me.

<!--more-->

To briefly introduce the two approaches, they have very different main goals:
* tactical: get something working
* strategic: produce a great design, which also happens to work.

While most engineers can easily agree that strategic programming is the better one in an ideal world, we also know that the main challenge to adapt is how to compete with other priorities.
The author's solution is to contribute about 10-20% of your total development time on contiguous design improvement.

I love this investment mindset: it will put a speed bump on the speed of complexity growth, and very likely it gets to pay off some existing tech debts.
It avoids two major issues about addressing tech debts:
1. refactor work always has a hard time to compete with feature work, and
2. a 20% upper-bound prevents huge up-front investment to design the entire system (which you can argue this is the traditional waterfall method).

What can the 10-20% time be used on? There are many options:
1. Try alternative designs before committing to a (likely first) idea.
2. Refactor existing code and tests.
3. Propose better designs.
4. Write comments/documents for changes.
5. Raise existing design issues for broader awareness.
...

Of course, there are still challenges for adoption that I can imagine. First of all, you still need to convince leadership that it's okay to increase the time budget of every project by 10-20%. Secondly, you might still need some extra incentives for engineers so that they allocate the extra time on the areas you want. However, I still think this is an approach I should try out. It will make me work slower in a short term, but if I want to elevate myself to a role of force-multiplier then having a good idea of how to improvement our system will be a great asset.
