---
layout: post
title: Statistical Tests for Dumb Walkers
date: 2025-05-13 01:00:00
description: How to see if 
tags: calculus linear-algebra matrix-calculus
categories: math
---

<style>
        .center-box {
            background-color: #ccc;
            padding: 20px;
            text-align: center;
            border-radius: 8px;
            width: 50%;
        }
 </style>


I'm a big walker, and lately one of my favorite walks has been from 

Lately, I've noticed a strange


But there's a problem&mdash;while I'm going on these walks, I'm daydreaming, thinking about random things, calling friends and family, or so on. And even if I wasn't doing any of those things, I have a tendency to forget a 




Let's 


# Modelling the Process

Let's denote by $S_t$ the running average of my walk, after taking $t$ samples. Of course, I don't know $t$---I just know that, whatever $t$ I'm on, I 

$$
\mathbb{E}[S_{t+1} \mid S_t] = p \cdot (\mathbb{E}[S_t] + 1) + q \cdot (\mathbb{E}[S_t] - 1) = \mathbb{E}[S_t] + (p - q).
$$

Under the null hypothesis, $p=q=1/2$, and so $S_t$ is a martingale. Intuitively, we think that this

---

#### **Theorem (Ville's Inequality)**



---

This is a general fact (which extends way beyond real vector spaces), but it has a really nice implication for us. It turns out, we can actually define the gradient relative to this representation:

---




In any case, thanks for reading.

&mdash;Thomas