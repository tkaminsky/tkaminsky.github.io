---
layout: post
title: Lyapunov, Control Lyapunov, and Control Barrier Functions
date: 2024-11-04 10:16:00
description: A brief introduction to Lyapunov theory as I currently understand it.
tags: lyapunov control dynamical-systems
categories: control
---

Lately, I've been running into a lot of work that uses Lyapunov functions to prove statements about the convergence of certain distributed algorithms. I have a limited background in control, and so I initially struggled to wrap my head around these ideas. I also found that it wasn't super easy to find all of the information that I wanted in a digestible format online. To hopefully begin to remedy this, here is my attempt to give a commonsense explanation of Lyapunov Functions, Control Lyapunov Functions, and Control Barrier Functions, at a level relevant to multi-robot coordination research.

This blog post is definitely a work in progress. I'm writing down stuff as I learn it, and at times I include proofs or intuition which aren't in the source papers, and as such may posess errors. That said, if you have any corrections or other ideas, please feel free to contact me at ```tkaminsky@g.harvard.edu```.

### Table of Contents
* **[Part I: Lyapunov Functions](#part-i--lyapunov-functions)**
* **[Part II: Control Lyapunov Functions](#part-ii-control-lyapunov-functions)**
* **[Part III: Control Barrier Functions](#part-iii--control-barrier-functions)**
* **[Part IV: Controlling with CBFs and CLFs](#part-iv--controlling-with-cbfs-and-clfs)**
* **[References and Further Reading](#references--further-reading)**


# Part I : Lyapunov Functions

Throughout this blog post, we're working in the world of dynamical systems&mdash;that is, we assume that we have access to some system dynamics of the form

$$
\overset{.}{x}(t) = F(x(t)) \;\; \forall t \in [0,\tau).
$$

In particular, we care about these dynamics when $x$ is in some neighborhood around $0$, which we define as a set <span class="inlinecode">$\mathcal{D} \in \mathbb{R}^n$</span> such that $0 \in \mathcal{D}$. Let's engineer $F$ so that $0$ is an *equilibrium point*, so

$$
F(0) = 0.
$$

This means that if $x(t) = 0$, then it will stay there forever, since its time derivative $F(0)=0$. Notice that if some other point $x^{\*}$ is an equilibrium point, we can always just construct a shifted dynamics $F'(x) = F(x - x^\*)$ so that $0$ is the equilibrium instead, so we just assume for the sake of simplicity that we've already shifted the equlibrium to 0. Then our goal is to find out whether this system is *stable*&mdash;that is, whether trajectories $x(t)$ stay close to equilibrium, converge to equlibrium, or converge *quickly* to equilibrium.

[Aleksandr Lyapunov](https://en.wikipedia.org/wiki/Aleksandr_Lyapunov) (1857-1918) was a Russian mathematician who did a lot of work answering these questions. He defined a a few important notions of stability, including the following three:

* **Lyapunov Stability**: $x(t) = 0$ is Lyapunov stable if, for all $\varepsilon > 0$, there exists $\delta > 0$ such that $\norm{x(0)} < \delta \implies \norm{x(t)} < \varepsilon \;\; \forall t$. 
    * Intuitively, a lyapunov-stable solution will stay less than $\varepsilon$-far away from the equlibrium point if we place it $\delta$-far away at initialization.


* **Asymptotic Stability**: $x(t) = 0$ is Asymptotically stable if it is Lyapunov stable and, for some $\delta > 0$, if $\norm{x(0)} < \delta$, then $\underset{t \to \infty}{\text{lim}} x(t) = 0$. 
    * That is, in addition to staying 'close' to equilibrium, our trajectories actually reach equilibrium in the limit, assuming they start out reasonably close to it.


* **Exponential Stability**: $x(t) = 0$ is exponentially stable if there are some $\alpha, \beta, \delta > 0$ such that, if $\norm{x(0)} < \delta$, then $\norm{x(t)} \leq \alpha \norm{x(0)}e^{-\beta t}$ for all $t$. 
    * This looks like asymptotic stability, under the additional constraint that it converges quickly (exponentially-fast) to equilibrium.

So Lyapunov stability just says that $x(t)$ won't get 'too far away' from $0$, asymptotic stability says that $x(t)$ will eventually reach exactly $x(t) \to 0$, and exponential stability says that $x(t) \to 0$ at an exponential rate.


One of the big ideas from Lyapunov theory is that, if we want to figure out whether an equilibrium point satisfies one of these stability conditions, we don't need to directly look at trajectories $\{x(t)\}$. Consider for example a pendulum, with equilibrium corresponding to the pendulum being at rest. One way to show that this equilibrium point is (Lyapunov) stable would be to consider the *mechanical energy* of the system&mdash;the sum of the system's kinetic and potential energies&mdash;and show that *it* is bounded. In particular, we know that the mechanical energy of a closed system must be constant. Thus, if we take the maximum height of our pendelum to be the point where all of the mechanical energy is converted to potential energy (we swing as high as possible, until we have 0 velocity), we know that our pendelum can never swing higher! Thus, we have bounded the possible displacement from equilibrium for our pendelum system.


Lyapunov generalizes this idea of the mechanical energy of a system by defining so-called **Lyapunov Functions**. The definition is very general, but in practice this can almost always be thought of as an 'energy' at each point in a trajectory:

***

#### **Definition (Lyapunov Function):**
Let $\overset{.}{x}(t) = F(x(t)) $ be a dynamical system with $x(t)=0$ a fixed point, as defined above. Then a continuously differentiable function $V : \mathcal{D} \to \mathbb{R}$ is called a **Lyapunov Function** for the system if the following hold:

1. $V(0) = 0$.
2. $V(x) > 0 \;\; \forall x \in \mathcal{D} \setminus \\{0\\}$.
3. The directional derivative $D_xV(x) \cdot \overset{.}{x}(t) = D_x V(x) \cdot F(x(t)) \leq 0$.

***

Notice how these properties coincide with our understanding of energy. The first two require that $V$ is only $0$ at equilibrium, and otherwise positive&mdash;this reflects our intuition that a trajectory 'at rest' in a stable equilibrium should have no kinetic energy and potential energy which could be defined as 0. Any displacement from equilibrium should either result in changing velocity and/or reaching a higher potential energy, so we'd expect non-equilibrium positions to have positive mechanical energy. The final condition thus says that $x(t)$ is always moving towards a lower-energy state (or, at least, not moving to a higher-energy state). If this is true, then we know that our position is never 'gaining energy', and so it can only get so far away from its current position.

Using Lyapunov functions, we can characterize the stability of equilibrium point. I won't prove this result (see Haddad et al.'s [textbook](https://www.jstor.org/stable/j.ctvcm4hws), p. 137-140, for details), but the following is an important theorem of Lyapunov:

***
#### **Theorem (Lyapunov Stability).**

Let $\overset{\cdot}{x}(t) = F(x(t))$ be a dynamical system defined as above. Then the following hold:

1. If A Lyapunov function $V$ exists for fixed point $0$, then $x(t) = 0$ is *Lyapunov stable*.

2. If furthermore $$D_x V(x) \cdot F(x(t)) < 0,$$ then the equilibrium is *asymptotically stable*.

3. Finally, if there are also $\alpha, \beta, \varepsilon >0$, $p \geq 1$ such that 

    * $\alpha \norm{x}^p \leq V(x) \leq \beta \norm{x}^p$, and
    * $D_xV(x) \cdot F(x(t)) \leq -\varepsilon V(x)$,

    then the point is *exponentially stable*.

***

Thus, we see that stability can be determined by considering properties of the Lyapunov function, rather than by looking at the trajectory directly! There's more to say about this, but it seems as though basic Lyapunov theory is pretty well-covered online already. Especially helpful sources for me were Haddad et al.'s textbook [Nonlinear Dynamical Systems and Control: A Lyapunov-Based Approach](https://www.jstor.org/stable/j.ctvcm4hws), Slotine and Li's textbook [Applied Nonlinear Control](https://lewisgroup.uta.edu/ee5323/notes/Slotine%20and%20Li%20applied%20nonlinear%20control-%20bad%20quality.pdf), and this [blog post](https://byjus.com/maths/lyapunov-functions/).

However, there is one final point which is worth thinking about before we move on. In particular, asymptotic stability only requires that there exists *some* ball $B_\delta(0)$ in which every trajectory converges to 0. It isn't obvious exactly which balls work for this, and this matters a lot in later sections, when we want to make sure that our system will converge to the equilibrium from a broad range of starting positions.

To start to get at this, note that condition (3) of the Lyapunov Function definition implies that the derivative $\frac{d V(x(t))}{dt}$ is non-positive. In other words, $V(x(t))$ is a nonincreasing function. Then we can use the fact that $V(x(t)) \leq V(x(0))$ for all $t$ to make the following statement:

***

#### **Lemma : Invariance of Sublevel Sets of Lyapunov Functions**

Consider any Lyapunov function $V$, and a corresponding *sublevel set* $S_\beta = \\{ x \in \mathbb{R}^n \| V(x) \leq \beta \\}$ satisfying $S_\beta \subseteq \mathcal{D}$. Then $S_\beta$ is *forward-invariant*. That is, if $x(0) \in S_\beta$, then $x(t) \in S_\beta $ for all $t$.

Moreover, if property $(2)$ of Lyapunov's Stability Theorem holds, then $x(t) \to 0$ for all $x(0) \in S_\beta$ if either:

1. $\mathcal{D}$ is bounded, or
2. $V(x)$ is *radially unbounded*&mdash;that is, as $\norm{x}\to\infty$, $V(x) \to \infty$.

***

**_Proof_**.

Say that this statement were false. Then, since the trajectory $x(t)$ is continuous, there must exist some time $s$ such that $x(s)$ is not in $S_\beta$. Then, since by construction $V(x(0)) < \beta$ and $V(x(s)) > \beta$, we find

$$
\frac{
V(x(s)) - V(x(0))
}
{
    s - 0
}
:= m
>
\frac{\beta - \beta}{s}
=
0.
$$

Thus, there is some positive secant slope $m$ connecting these two points. But since $V$ and $x$ are both continuously differentiable by construction, $V(x(t))$ is as well, and so we can apply the mean value theorem to find that there must be some time $s_0$ such that

$$
\frac{d V(x(t))}{dt}
=
D_xV(x) \cdot F(x(t)) = m > 0,
$$

which contradicts our definition of a Lyapunov function. Thus, we find that each $S_\beta$ must be forward-invariant.

Consider then the second condition (I'm still working on this part: Come back later!).



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

is a dynamical system which can be analyzed using the usual Lyapunov functions from [Part I](#part-i--lyapunov-functions). However, this methodology has some major problems. Most pressingly, if it's *hard* to randomly pick a $u(x)$ that stabilizes the system, then we have no way to figure out 'good' choices of $u$ other than just by plugging in a ton of options. This motivates Control Lyapunov Functions&mdash;we want to create an object which can allow us to *locate* good controls for a given problem.

We formally define a Control Lyapunov Function as follows:

***

#### **Definition (Control Lyapunov Function (CLF)):**

Let $F(x(t), u(t))$ be a controlled dynamical system defined as above. Then a continuously differentiable function $V: \mathcal{D} \to \mathbb{R}$ is called a control Lyapunov function for the system if the following hold:

1. $V(0) = 0$.
2. $V(x) > 0 \;\; \forall x \in \mathcal{D} \setminus \{0\}$.
3. $ \underset{u \in \mathcal{U}}{\inf} \; D_xV(x) \cdot  F(x, u) < 0$ for all $x \in \mathcal{D} \setminus \{0\}$.

***

The first two conditions are identical to a normal Lyapunov function. The third is a little stranger. It requres that, for every point $x$, there exists some control input $u$ that causes the directional derivative to be negative. We can think of this as implying the existence of a stabilizing feedback controller. In particular, taking

$$
u^*(x) \in \{u \in \mathcal{U} | D_xV(x) \cdot F(x,u) < 0\} \;\; \forall x \in \mathcal{D},
$$

where each of these sets is guaranteed to be nonempty by condition (3), we find that the dynamical system

$$
\overset{.}{x}(t) = F(x(t), u^*(x(t)))
$$

is asymptotically stable, since $V$ is a Lyapunov function satisfying criterion (2) from Lyapunov's Stability Theorem. Conversely, if condition (3) is not met, then there must be some $x$ such that NO control can stabilize it, and so there does not exist a stable controller. Thus, we can interpret the existence of a CLF as a necessary and sufficient condition for the dynamical system to be asymptotically stabilizable using a feedback controller. We summarize this in the following theorem:

***

#### **Theorem (Stability of Controlled Systems)**

Let $\overset{.}{x}(t) = F(x(t),u)$ be a controlled dynamical system defined as above. Then this system is asymptotically stabilizable by a feedback controller if and only if there exists a Control Lyapunov Function for the system.

***

Now, in some sense, we've solved our problem of characterizing optimal controllers. If we can develop an algorithm which can sample members of $\\{u \in \mathcal{U} \| D_xV(x) \cdot F(x,u) < 0\\}$, then we can use this scheme to create a controller. However, it is not always clear how to do this in practice. In the following section, we identify a strategy for calculating optimal controllers in the special case of a linear controller.


### Special Case : Linear Control

Assume that we are working with an *affine control system* governed my the dynamics

$$
\overset{.}{x}(t)
=
F(x(t))
+
G(x(t)) u(t) \;\; \forall t \in [0,\tau)
$$

for some $F : \mathbb{R}^n \to \mathbb{R}^n$ and $G: \mathbb{R}^n \to \mathbb{R}^{n \times m}$, both smooth. Assume also for the time being that the control space $$\mathcal{U} = \mathbb{R}^m$$. For a simple example of such a system, say that $G(x) = I$ for all $x$. Then at each time our control input corresponds to moving the agent in any direction.

In this setting, we can make a stronger statement about the existence of optimal control:

***

#### **Theorem (Affine Control Lyapunov Function)**

Under an affine control system $\overset{.}{x}(t) = F(x(t)) + G(x(t)) u(t)$ defined as above, a positive-definite, radially unbounded function $V:\mathbb{R}^n \to \mathbb{R}$ is a Control Lyapunov Function if and only if

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

Consider condition (3) of a CLF, that is:

$ \underset{u \in \mathcal{U}}{\inf} \; D_xV(x) \cdot  F(x, u) < 0$ for all $x \in \mathcal{D} \setminus \\{0\\}.$

By plugging in the affine control to this expression, we see that it's equivalent to 

$$
\underset{u \in \mathcal{U}}{\inf} \; D_xV(x) \cdot [F(x(t))
+
G(x(t)) u(t)]
$$

$$
= \underset{u \in \mathcal{U}}{\inf} \; 
\left[
D_xV(x) \cdot F(x(t))
+
D_xV(x) \cdot 
G(x(t)) u(t)
\right].
$$

If $D_xV(x) \cdot 
G(x(t)) \neq \mathbf{0}$, then there must be some non-zero entry at index $i$ with sign $\varepsilon$. Then taking $u_i \to -\varepsilon \cdot \infty$, we see that the entire expression can approach $-\infty < 0$.

Thus, the only conditions that we care about occur when $D_xV(x) \cdot 
G(x(t)) = \mathbf{0}$&mdash;that is, for points in $\mathcal{R}$. In these cases, we require that

$$
\underset{u \in \mathcal{U}}{\inf} \; 
D_xV(x) \cdot F(x(t))
<
0,
$$

which is exactly our result. 


With this lemma, we can explicitly design a controller which globally (for any $x \in \mathbb{R}$ stabilizes our system). We define the following controller:

***

#### **Theorem : Linear Feedback Controller**

For the affine control system $\overset{.}{x}(t)=F(x(t))+G(x(t)) u(t)$ defined above, there exists a feedback controller $\phi : \mathbb{R}^n \to \mathbb{R}^m$ given by the following:

$$
\phi(x)
=
\begin{cases}
    - \left( 
        c_0 + \frac{\alpha(x) + \sqrt{\alpha^2(x) + \left[ \beta^T(x) \beta(x) \right]^2}}{\beta^T(x) \beta(x)}
    \right) \beta(x), & \beta(x) \neq 0, \\
    0 & \beta(x) = 0,
\end{cases}
$$

where $c_0 \geq 0$ is any scalar,

$$
\alpha(x) := D_xV(x) \cdot F(x),
$$

and

$$
\beta(x) := 
\left[ 
D_xV(x) \cdot G(x)
\right]^T.
$$

***


**_Proof._**

We just plug into

$$
D_xV(x) \cdot [F(x) + G(x) \phi(x)]
=
\alpha(x) + \beta^T(x) \phi(x).
$$

If $\beta(x) = 0$, then we find $D_xV(x) = D_xV(x) \cdot F(x) < 0$ by the Affine Control Lyapunov Theorem.

Otherwise, we substitute in $\phi$ to find

$$
= \alpha(x) - \beta^T(x) \beta(x)
\left(  
    c_0 + 
    \frac{\alpha(x) + \sqrt{\alpha^2(x) + 
\left[ \beta^T(x) \beta(x) \right]^2}}{\beta^T(x) \beta(x)} \right)
$$

$$
= -\beta^T(x) \beta(x) c_0
-
\sqrt{\alpha^2(x) + 
\left[ \beta^T(x) \beta(x) \right]^2}
< 0,
$$

since both of these terms are negative by construction. Thus, we find that the derivative is negative, and so this is indeed a stabilizing controller.

But what's happening here intuitively? Consider for simplicity when $c_0 = 0$.

Then let's consider two examples. First, say that $G(x) = I$; that is, we just move in direction $u$. Then $\beta(x) = D_xV(x)$, so our control input has us move in the direction of steepest descent in $V$. This looks like a gradient descent! We have some 'strength' given by the big constant term, which we then apply to the direction which moves us towards equilibrium the fastest. Notice that our construction ensures that our derivative $\frac{d V(x(t))}{dt}$ is $-\sqrt{\alpha^2(x) + \norm{\beta}^4}$. So we see that this is the rate at which we will move down $V$.

As another example, say that we have a more constrained $G$ given by $G(x) = [1,0]^T$. Then our control input is a one-dimensional displacement in the $x$ axis. Then $\beta(x) = \frac{d V(x)}{d x_1}$. Thus, our optimal control has us moving in the direction that decreases $V$ as much as possible, given that we  can only control along the $x$ axis. Namely, if the partial along the $x$ axis is positive, we move in the negative direction, and vice-versa. Thus, we see that this looks like a constrained version of the previous example, 


Thus, we find that we can create optimal feedback controllers for affine systems.

### Note : Local Stabilizability

One question that seems relevant is, what if we can't assume that $\mathcal{U} = \mathbb{R}^m$? I'll fill this in later (Section TBD), but I think that the main idea is that obviously global stability could fail, because we might get too far away from the equilibrium to reasonably control there. However, if we are operating in a bounded set $\mathcal{D}$, then under certain regularity assumptions (like that everything we care about is Lipschitz on $\mathcal{D}$), we can bound the size of the optimal controller given above. Thus, we can create a controller constrained to $\mathcal{U}$ which is optimal, and perhaps we can even say the 'weakest' controller which can still work!







# Part III : Control Barrier Functions

Thus far, we have mostly been concerned with studying the *stability* of equilibria and controlled equilibria. However, there are often other concerns worth accounting for during planning.

In this section, we consider *safety* concerns in dynamical systems. In particular, it is possible that some $x(t)$ are 'bad' states&mdash;for example, some orientations of a robotic arm could cause it to hit an object, or an autonomous vehicle being in certain regions of space could be dangerous if those regions are also occupied by humans. In general, we could imagine assigning a 'badness' score to every point in our state space, with negative badness scores being avoided. Then we could cast the problem of ensuring safety as enforcing that we remain in 'good' zones; that is, those with positive score.

This idea enables us to define a *Barrier Function* (technically, a Zeroing Barrier Function, see [Ames et al.](https://ieeexplore.ieee.org/abstract/document/7782377?casa_token=Su26OE1ZV1EAAAAA:KMlvhbhQq0wb76ixit1QMQgIVqZEQj8Vf4fTlfewe5_auWi48Zb-nSSJ7mxopBuDgF0bLT3BHw)):

***

#### **Definition : Barrier Function**

A continuously differentiable function $h : \mathcal{D} \to \mathbb{R}$ is called a *Barrier Function* for $\mathcal{C} = \\{ x\in \mathbb{R}^n \| h(x) \geq 0 \\}$ if $\mathcal{C} \subseteq \mathcal{D}$ and there exists some strictly increasing function $\alpha : \mathbb{R} \to \mathbb{R}$ with $\alpha(0) = 0$ such that

$$
\frac{d h(x(t))}{d t}
=
D_xh(x) \cdot F(x) \geq -\alpha(h(x)).
$$

***

This definition implies that, at the boundary $\partial \mathcal{C} = \\{x \in \mathbb{R}^n \| h(x) = 0 \\}$, 

$$
\frac{d h(x(t))}{d t}
\geq -\alpha(0) = 0,
$$

so $h(x)$ will not leave $\mathcal{C}$ if it is already inside it. Thus, we see that this definition enforces forward invariance as we desire. In fact, we can actually get an even stronger condition. Decause $\alpha$ is strictly increasing, if we are outside of our set $\mathcal{C}$, then we find that

$$
\frac{d h(x(t))}{d t}
\geq -\alpha(0) > 0,
$$

so we will be travelling towards $\mathcal{C}$ whenever we are outside of it. This can be formalized as a Lyapunov function as follows:

***

#### **Theorem : BF to Lyapunov Function**

Say that $h(x)$ is a Barrier Function for $\mathcal{C}$ with domain $\mathcal{D}$. Then define $V_\mathcal{C}$ as follows:

$$
V_\mathcal{C}(x)
=
\begin{cases}
    0, & \text{ if } x \in \mathcal{C},\\
    -h(x) & \text{ if } x \in \mathcal{D} \setminus \mathcal{C}.
\end{cases}
$$

$V_\mathcal{C}$ is a Lyapunov function for our problem. Moreover, $\mathcal{C}$ is an asymptotically stable fixed set.

***

Here we generalize a Lyapunov function to a set, rather than a particular point&mdash;this causes no real issues, but it is a worthwhile exercise to make sure that everything still makes sense when we do this replacement.

It actually turns out that the converse of this theorem is also true if $\mathcal{C}$ is nonempty and compact. That is, we also have that $\mathcal{C}$ is a forward-invariant set if and only if you can create a barrier function for it!

However, Barrier Functions don't allow us to steer our dynamical system to safe regions. To do this, assume now that we are in an affine control system

$$
\overset{.}{x}(t)
=
F(x(t))
+
G(x(t)) u(t) \;\; \forall t \in [0,\tau)
$$


 analogously to defining CLFs from LFs, we define the notion of a *Control Barrier Function*:


***

#### **Definition : Control Barrier Function**

A continuously differentiable function $h : \mathcal{D} \to \mathbb{R}$ is called a *Control Barrier Function* for $\mathcal{C} = \\{ x\in \mathbb{R}^n \| h(x) \geq 0 \\}$ if $\mathcal{C} \subseteq \mathcal{D}$ and there exists some strictly increasing function $\alpha : \mathbb{R} \to \mathbb{R}$ with $\alpha(0) = 0$ such that

$$
\underset{u \in \mathcal{U}}{\text{sup}}
\frac{d h(x(t))}{d t}
=
\underset{u \in \mathcal{U}}{\text{sup}}
D_xh(x) \cdot F(x) + D_x h(x) \cdot G(x) u \geq -\alpha(h(x)).
$$

***

This looks really similar to our definition of the CLF. Now, we see that there must exist some control input that will render each induce a barrier function when the corresponding control is substituted in. The analysis here is exactly the same as in the CLF case (indeed, we can define a CLF as in the previous section as the negation of our CBF), so we omit it here. The important thing is that we can design a control which allows us to ensure that we remain in a safe set.


# Part IV : Controlling With CBFs and CLFs

Let's tie this all together by defining concrete

First, say that we are given a target feedback controller $\phi(x)$. We want to derive a controller $u$ which is as close to $\phi$ as possible, but with the added constraint that it must be safe with respect to some set $\mathcal{C}$. We can get this by solving the [Quadratic Program](https://en.wikipedia.org/wiki/Quadratic_programming) (QP):

$$
\begin{align*}
u(x) &= \underset{u \in \mathcal{U}}{\text{arg min}} \frac{1}{2} \norm{u - \phi(x)}^2 \\
& \text{ s.t. } D_xh(x) \cdot F(x) + D_x h(x) \cdot u \geq -\alpha (h(x)).
\end{align*}.
$$

According to work outside the scope of this blog post (see [Ames et al., 2019.](https://coogan.ece.gatech.edu/papers/pdf/amesecc19.pdf)), this problem has a closed-form solution when $\mathcal{U} = \mathbb{R}^n$.

More generally, we can use this strategy to assign a controller that both stabilizes the system (as determined by some CLF $V$) AND offers safety guaranteed by $h$. In particular, the following QP works:

$$
\begin{align*}
u(x) &= \underset{(u, \delta) \in \mathbb{R}^{m+1}}{\text{arg min}} \frac{1}{2} u^T H(x) u + p \delta^2 \\
& \text{ s.t. } D_x V(x) \cdot F(x) + D_x V(x) \cdot u \leq -\gamma (V(x)) + \delta\\
& D_xh(x) \cdot F(x) + D_x h(x) \cdot u \geq -\alpha (h(x)),
\end{align*}
$$

where $\gamma$ is another class $\mathcal{K}$ function, $H(x)$ is any positive definite matrix (so that term drives $\norm{u} \to 0$) and $\delta$ is a relaxation parameter which ensures that the program is solvable. Having the term $p \delta^2$ in the constraint drives this relaxation to $0$. The solution to this problem is guaranteed to be safe, and is likely to be stable (more so as we take $\delta \to 0$). 


# Conclusion


Thus, we find that the problem of controlling subject to some safety constraint, or optimally controlling with both a stability and safety objective, can be achieved using CLFs and CBFs, and solved using existing tools for Quadratic Programming. For example, see the [Control Barrier Function Toolbox](https://www.ll.mit.edu/partner-us/available-technologies/control-barrier-function-toolbox) from Lincoln Labs.

It is perhaps worthwhile to see some concrete examples of CLFs and CBFs in the wild. I plan to cover this in a future blog post, focusing in particular on a paper out of my lab ([Cavorsi et al., 2023. ](https://ieeexplore.ieee.org/abstract/document/10354416?casa_token=6jyiwNV7sCEAAAAA:5BLuAUeRw1ZmuZaxHnD_YfWKkL0wZqbVP8pHQNU8xamrJAbb9cJMJGvkxzNIYrNsIZ59Or0)).

In any case, thanks for reading.

&mdash;Thomas







# References + Further Reading

Here are a few of the books and papers that I found especially helpful. Especially clear writing is marked in **bold**.

#### Textbooks:

* [Slotine and Li, 1991. Applied Nonlinear Control](https://lewisgroup.uta.edu/ee5323/notes/Slotine%20and%20Li%20applied%20nonlinear%20control-%20bad%20quality.pdf)

* **[Haddad and Chellaboina, 2008. Nonlinear Dynamical Systems and Control: A Lyapunov-Based Approach](https://www.jstor.org/stable/j.ctvcm4hws)**

#### Papers (Chronological):

* [Artstein, 1982. Stabilization with Relaxed Controls](https://www.sciencedirect.com/science/article/pii/0362546X83900494)

* [Sontag, 1983. A Lyapunov-Like Characterization of Asymptotic Controllability](https://epubs.siam.org/doi/abs/10.1137/0321028?casa_token=UBCu6QQcDNQAAAAA:gnQ5sEIIyS-uXT8bzuob6UIffSQjAbhfVmpw8qx_cxOJ-RmMJT25gE1Hb5iwxCJqXlYsIl9J)

* [Sontag, 1989. A 'Universal' Construction of Artstein's Theorem on Nonlinear Stabilization](http://www.sontaglab.org/FTPDIR/art-sycon8903.pdf)

* **[Ames et al., 2016. Control Barrier Function Based Quadratic Programs for Safety Critical Systems](https://ieeexplore.ieee.org/abstract/document/7782377?casa_token=Su26OE1ZV1EAAAAA:KMlvhbhQq0wb76ixit1QMQgIVqZEQj8Vf4fTlfewe5_auWi48Zb-nSSJ7mxopBuDgF0bLT3BHw)**

* [Ames et al., 2019. Control Barrier Functions: Theory and Applications](https://coogan.ece.gatech.edu/papers/pdf/amesecc19.pdf)

* [Capelli et al., 2020. Connectivity Maintenance: Global and Optimized Approach Through Control Barrier Functions](https://ieeexplore.ieee.org/abstract/document/9197109?casa_token=HEt2jergrKEAAAAA:oStQJxHycKbc4iCHloCpSi62P8oWoPqILA4k_tcUEYNrMS76EK40c4lyLcJSjdBnA5LlL7E)

* [Cavorsi et al., 2023. Multirobot Adversarial Resilience Using Control Barrier Functions](https://ieeexplore.ieee.org/abstract/document/10354416?casa_token=6jyiwNV7sCEAAAAA:5BLuAUeRw1ZmuZaxHnD_YfWKkL0wZqbVP8pHQNU8xamrJAbb9cJMJGvkxzNIYrNsIZ59Or0)





<!---
https://www.ll.mit.edu/partner-us/available-technologies/control-barrier-function-toolbox
-->