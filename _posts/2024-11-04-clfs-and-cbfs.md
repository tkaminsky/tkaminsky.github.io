---
layout: post
title: Lyapunov Functions and Control Lyapunov Functions
date: 2024-11-04 10:16:00
description: A brief introduction to Lyapunov theory as I currently understand it.
tags: lyapunov control dynamical-systems
categories: sample-posts
---

Lately, I've been running into a lot of work that uses Lyapunov functions to prove statements about the convergence of certain distributed algorithms. I have a limited background in control, and so I initially struggled to wrap my head around these ideas. I also found that it wasn't super easy to find all of the information that I wanted in a digestible format online. To hopefully begin to remedy this, here is my attempt to give a commonsense explanation of Lyapunov Functions, Control Lyapunov Functions, and Control Barrier Functions (next post), at a level relevant to multi-robot coordination research.

### Table of Contents
* **[Part I: Lyapunov Functions](#part-i--lyapunov-functions)**
* **[Part II: Control Lyapunov Functions](#part-ii-control-lyapunov-functions)**


# Part I : Lyapunov Functions

Throughout this blog post, we're working in the world of dynamical systems&mdash;that is, we assume that we have access to some system dynamics of the form

$$
\overset{.}{x}(t) = F(x(t)) \;\; \forall t \in [0,\tau).
$$

In particular, we care about these dynamics when $x$ is in some neighborhood around $0$, which we define as a set $\mathcal{D} \in \mathbb{R}^n$ such that $0 \in \mathcal{D}$. Let's engineer $F$ so that $0$ is an *equilibrium point*, so

$$
F(0) = 0.
$$

This means that if $x(t) = 0$, then it will stay there forever, since its time derivative $F(0)=0$. Notice that if some other point $x^*$ is an equilibrium point, we can always just construct a shifted dynamics $F'(x) = F(x - x^*)$ so that $0$ is the equilibrium instead, so we just assume for the sake of simplicity that we've already shifted equlibrium to 0. Then our goal is to find out whether this system is *stable*&mdash;that is, whether trajectories $x(t)$ stay close to equilibrium, converge to equlibrium, or converge *quickly* to equilibrium.

[Aleksandr Lyapunov](https://en.wikipedia.org/wiki/Aleksandr_Lyapunov) (1857-1918) was a Russian mathematician who did a lot of work answering these questions. He defined a a few important notions of stability, including the following three:

* **Lyapunov Stability**: $x(t) = 0$ is Lyapunov stable if, for all $\varepsilon > 0$, there exists $\delta > 0$ such that $||x(0)|| < \delta \implies ||x(t)|| < \varepsilon \;\; \forall t$. 
    * Intuitively, a lyapunov-stable solution will stay less than $\varepsilon$-far away from the equlibrium point if we place it $\delta$-far away at initialization.

* **Asymptotic Stability**: $x(t) = 0$ is Asymptotically stable if it is Lyapunov stable and, for some $\delta > 0$, if $||x(0)|| < \delta$, then $\underset{t \to \infty}{\text{lim}} x(t) = 0$. 
    * That is, in addition to staying 'close' to equilibrium, our trajectories actually reach equilibrium in the limit, assuming they start out reasonably close to it.

* **Exponential Stability**: $x(t) = 0$ is exponentially stable if there are some $\alpha, \beta, \delta > 0$ such that, if $||x(0)|| < \delta$, then $||x(t)|| \leq \alpha ||x(0)||e^{-\beta t}$ for all $t$. 
    * This looks like asymptotic stability, under the additional constraint that it converges quickly (exponentially-fast) to equilibrium.

So Lyapunov stability just says that $x(t)$ won't get 'too far away' from $0$, asymptotic stability says that $x(t)$ will eventually reach exactly $x(t) \to 0$, and exponential stability says that $x(t) \to 0$ at an exponential rate.


One of the big ideas from Lyapunov theory is that, if we want to figure out whether an equilibrium point satisfies one of these stability conditions, we don't need to directly look at trajectories $\{x(t)\}$. Consider for example a pendulum, with equilibrium corresponding to the pendulum being at rest. One way to show that this equilibrium point is (Lyapunov) stable would be to consider the *mechanical energy* of the system&mdash;the sum of the system's kinetic and potential energies&mdash;and show that *it* is bounded. In particular, we know that the mechanical energy of a closed system must be constant. Thus, if we take the maximum height of our pendelum to be the point where all of the mechanical energy is converted to potential energy (we swing as high as possible, until we have 0 velocity), we know that our pendelum can never swing higher! Thus, we have bounded the possible displacement from equilibrium for our pendelum system.


Lyapunov generalizes this idea of the mechanical energy of a system by defining so-called **Lyapunov Functions**. The definition is very general, but in practice this can almost always be thought of as an 'energy' at each point in a trajectory:

***

### Definition (Lyapunov Function):
Let $\overset{.}{x}(t) = F(x(t)) $ be a dynamical system with $x(t)=0$ a fixed point, as defined above. Then a continuously differentiable function $V : \mathcal{D} \to \mathbb{R}$ is called a **Lyapunov Function** for the system if the following hold:

1. $V(0) = 0$.
2. $V(x) > 0 \;\; \forall x \in \mathcal{D} \setminus \{0\}$.
3. The directional derivative $D_xV(x) \cdot \overset{.}{x}(t) = D_x V(x) \cdot F(x(t)) \leq 0$.

***

Notice how these properties coincide with our understanding of energy. The first two require that $V$ is only $0$ at equilibrium, and otherwise positive&mdash;this reflects our understanding that, for an object to be at rest, it must require an input of energy to start moving, and, likewise, if there is some mechanical energy still in the system there must be some other state which it could move to that has a lower energy. The final condition thus says that $x(t)$ is always moving towards a lower-energy state (or, at least, not moving to a higher-energy state). If this is true, then we know that our position is never 'gaining energy', and so it can only get so far away from its current position.

Using Lyapunov functions, we can characterize the stability of equilibrium point. I won't prove this result (see Haddad et al.'s [textbook](https://www.jstor.org/stable/j.ctvcm4hws), p. 137-140, for details), but the following is an important theorem of Lyapunov:

***
### Theorem (Lyapunov Stability).

Let $\overset{\cdot}{x}(t) = F(x(t))$ be a dynamical system defined as above. Then the following hold:

1. If A Lyapunov function $V$ exists for fixed point $0$, then $x(t) = 0$ is *Lyapunov stable*.

2. If furthermore $$D_x V(x) \cdot F(x(t)) < 0,$$ then the equilibrium is *asymptotically stable*.

3. Finally, if there are also $\alpha, \beta, \varepsilon >0$, $p \geq 1$ such that 

    * $\alpha ||x||^p \leq V(x) \leq \beta ||x||^p$, and
    * $D_xV(x) \cdot F(x(t)) \leq -\varepsilon V(x)$,

    then the point is *exponentially stable*.

***

Thus, we see that stability can be determined by considering properties of the Lyapunov function, rather than properties of the trajectory directly! There's more to say about this, but it seems as though basic Lyapunov theory is pretty well-covered online already. Especially helpful sources for me were Haddad et al.'s textbook [Nonlinear Dynamical Systems and Control: A Lyapunov-Based Approach](https://www.jstor.org/stable/j.ctvcm4hws), Slotine and Li's textbook [Applied Nonlinear Control](https://lewisgroup.uta.edu/ee5323/notes/Slotine%20and%20Li%20applied%20nonlinear%20control-%20bad%20quality.pdf), and this [blog post](https://byjus.com/maths/lyapunov-functions/). For now, I'll instead move on to ideas which are a little less accessible online.

# Part II: Control Lyapunov Functions

For this part, we are going to tweak our setting slightly. Say now that we can apply a *control input* $u(t)$ which lets us effect the state $x(t)$&mdash;for example, say that we had a motor which could supply some force in a direction of our choosing at each timestep. Unless otherwise mentioned, we let $u : \mathcal{D} \to \mathcal{U} \subseteq \mathbb{R}^m$ be a *feedback control* for our system: that is, $u$ is a function of $x$, $u(x)$ is chosen to apply a correction at point $x$ which pushes it towards equilibrium, and $\mathcal{U}$ is the space of all possible inputs.

With this setup, we have a dynamical system which is a function of both the state and control input:

$$
\overset{.}{x}(t) = F(x(t), u(t)), \;\; \forall t \in [0, \tau).
$$

Our goal is now to figure out whether our control input will drive the system to equlibrium&mdash;or, more ambitiously, to somehow derive an optimal controller for this dynamical system.

One naive approach is to just choose some feedback controller $u(x)$ and let $\overset{\sim}{x} := [x,u(x)]$ be a new state vector. Then

$$
\overset{.}{\overset{\sim}{x}}(t) = \overset{\sim}{F}(\overset{\sim}{x}(t)) := F(x(t), u(x))
$$

is a dynamical system which can be analyzed using the usual Lyapunov functions from [Part I](#part-i--lyapunov-functions). However, this methodology has some major problems. Most pressingly, if it's *hard* to randomly pick a $u(x)$ that stabilizes the system, then we have no way to figure out 'good' choices of $u$ other than just by plugging in a ton of options. This motivates Control lyapunov Functions&mdash;we want to create an object which can allow us to *locate* good controls for a given problem.

We formally define a Control Lyapunov Function as follows:

***

### Definition (Control Lyapunov Function (CLF)):

Let $F(x(t), u(t))$ be a controlled dynamical system defined as above. Then a continuously differentiable function $V: \mathcal{D} \to \mathbb{R}$ is called a control Lyapunov function for the system if the following hold:

1. $V(0) = 0$.
2. $V(x) > 0 \;\; \forall x \in \mathcal{D} \setminus \{0\}$.
3. $ \underset{u \in \mathcal{U}}{\inf} \; D_xV(x) \cdot  F(x, u) < 0$ for all $x \in \mathcal{D} \setminus \{0\}$.

***

The first two conditions are identical to a normal Lyapunov function. The third is a little stranger. In common terms, it requres that, for every point $x$, there exists some control input $u$ that causes the directional derivative to be negative. We can think of this as implying the existence of a stabilizing feedback controller. In particular, taking

$$
u^*(x) \in \{u \in \mathcal{U} | D_xV(x) \cdot F(x,u) < 0\} \;\; \forall x \in \mathcal{D},
$$

where each of these sets is guaranteed to be nonempty by condition (3), we find that the dynamical system

$$
\overset{.}{x}(t) = F(x(t), u^*(x(t)))
$$

is asymptotically stable, since $V$ is a Lyapunov function satisfying criterion (2) from Lyapunov's Stability Theorem. Conversely, if condition (3) is not met, then there must be some $x$ such that NO control can stabilize it, and so there does not exist a stable controller. Thus, we can interpret the existence of a CLF as a necessary and sufficient condition for the dynamical system to be asymptotically stabilizable using a feedback controller. We summarize this in the following theorem:

***

### Theorem (Stability of Controlled Systems)

Let $\overset{.}{x}(t) = F(x(t),u)$ be a controlled dynamical system defined as above. Then this system is asymptotically stabilizable by a feedback controller if and only if there exists a Control Lyapunov Function for the system.

***

### Special Case : Linear Control

Oftentimes, we have more information about the form of our control than the general definition. In particular, we often assume that we are working with an *affine control system* governed my the dynamics

$$
\overset{.}{x}(t)
=
F(x(t))
+
G(x(t)) u(t) \;\; \forall t \in [0,\tau)
$$

for some $F : \mathbb{R}^n \to \mathbb{R}^n$ and $G: \mathbb{R}^n \to \mathbb{R}^{n \times m}$, both smooth. For a simple example of such a system, say that $G(x) = I$ for all $x$, and $u \in B_r(0) \subset \mathbb{R}^n$. Then at each time our control input lets us move the agent in any direction.

In this setting, we can make a stronger statement about the existence of optimal control:

***

### Theorem (Affine Control Lyapunov Function)

Under an affine control system $\overset{.}{x}(t) = F(x(t)) + G(x(t)) u(t)$ as defined above, $V:\mathbb{R}^n \to \mathbb{R}$ is a Control Lyapunov Function if and only if

$$
D_xV(x) \cdot F(x(t)) < 0 \;\; \forall x \in \mathcal{R},
$$

where

$$
\mathcal{R}
:=
\{x \in \mathbb{R}^n | x \neq 0, D_xV(x) \cdot G(x(t)) = \mathbf{0}\}.
$$

***

_**Proof.**_

Consider condition 3:

$ \underset{u \in \mathcal{U}}{\inf} \; D_xV(x) \cdot  F(x, u) < 0$ for all $x \in \mathcal{D} \setminus \{0\}.$

By plugging in the affine control to this expression, we see that it's equivalent to 

$$
\underset{u \in \mathcal{U}}{\inf} \; D_xV(x) \cdot [F(x(t))
+
G(x(t)) u(t)]
=
\underset{u \in \mathcal{U}}{\inf} \; 
\left[
D_xV(x) \cdot F(x(t))
+
D_xV(x) \cdot 
G(x(t)) u(t)
\right].
$$

if $D_xV(x) \cdot 
G(x(t)) \neq \mathbf{0}$, then there must be some non-zero entry at index $i$ with sign $\varepsilon$. Then taking $u_i \to -\varepsilon \cdot \infty$, we see that the entire expression can approach $-\infty < 0$.

Thus, the only conditions that we care about occur when $D_xV(x) \cdot 
G(x(t)) = \mathbf{0}$.

https://www.ll.mit.edu/partner-us/available-technologies/control-barrier-function-toolbox