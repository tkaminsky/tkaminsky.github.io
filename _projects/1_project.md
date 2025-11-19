---
layout: page
title: Intrinsic Norm Visualizer
description: An interactive tool for visualizing intrinsic norms of strongly convex functions.
img: assets/img/intrinsic-norm-visualizer/thumbnail.png
importance: 1
category: Personal
related_publications: true
---

- **[Background](#intrinsic-norm-visualizer)**
- **[The Visualizer](#the-visualizer)**

# Intrinsic Norm Visualizer

Last semester (Spring '25), I took a [course on optimization at MIT](https://www.mit.edu/~gfarina/67220/), taught by [Gabriele Farina](https://www.mit.edu/~gfarina/about/). Near the end of the course, we learned about self-concordant (SC) functions, which are a class of strongly-convex functions that have nice properties that make them suitable for optimization. One of the key concepts we explored was the intrinsic norm, which is a way to measure distances in the space defined by a SC function.

It yielded strange theorems, since often bounds were in terms of the intrinsic norm. For example, here's a remark from lecture 19 that I found strange:

---

#### **Remark (L19.3)**

If a point $x \in \Omega$ is such that $$\norm{n(x)}_x \leq 1/4$$, then there exists a minimum $z$ of $f$ within distance

$$
||z-x||_x \leq ||n(x)||_x + \frac{3 ||n(x)||_x^2}{(1-||n(x)||_x)^3}.
$$

---

Why the norm must be bounded by $1/4$, and why this number is scale-invariant, was not entirely obvious to me. I didn't have a good way to think of what being 'within $1/4$' looked like from the perspective of the intrinsic norm.

This tool is an effort to fix this, by seeing how the intrinsic norm 'appears' in both global and local coordinates. Please let me know if you have any thoughts for improvements.

# The Visualizer

To gain some visual intuition for what these intrinsic norms look like, I created [this website](https://tkaminsky.github.io/intrinsic-norm-visualizer/) for visualizing intrinsic norms of certain functions.

For the 6 given functions on top (only some of which are self-concordant), you can hover over a point to see the boundary of the 1-ball in the intrinsic norm centered at that point. Clicking will transform the linearly transform the function graph so all distances reflect those given by the intrinsic norm at that point. Thus, clicking on a point will show you what parts of the graph are 'close' in the intrinsic norm space. 

You can also click in the mini-graph on the top left to construct a polygon. This will automatically generate the barrier for that polygon, allowing you to see how this looks too.

I hope you enjoy.

&mdash;Thomas