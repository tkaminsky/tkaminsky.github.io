---
layout: post
title: Fun Linear Algebra Facts
date: 2026-11-04 10:16:00
description: A brief introduction to Lyapunov theory as I currently understand it.
tags: lyapunov control dynamical-systems
categories: control
---

I've been spending some time lately learning about matrix calculus, and it has made me realize how many really exciting and useful linear algebra identities are out there. Some of these come up naturally in course sequences in math, statistics, and computer science, but some are a bit less common. I thought that I'd spend some time here going over my favorites, and 

I've also included a PDF summarizing these facts, in case you are interested in a more concise 'cheat-sheet'.

### Table of Contents

- **[Part I: Lyapunov Functions](#part-i--lyapunov-functions)**
- **[Part II: Control Lyapunov Functions](#part-ii-control-lyapunov-functions)**
- **[Part III: Control Barrier Functions](#part-iii--control-barrier-functions)**
- **[Part IV: Controlling with CBFs and CLFs](#part-iv--controlling-with-cbfs-and-clfs)**
- **[References and Further Reading](#references--further-reading)**

# Part I : Lyapunov Functions


In any case, thanks for reading.

&mdash;Thomas

# References + Further Reading

Here are a few of the books and papers that I found especially helpful. Especially clear writing is marked in **bold**.

#### Textbooks:

- [Slotine and Li, 1991. Applied Nonlinear Control](https://lewisgroup.uta.edu/ee5323/notes/Slotine%20and%20Li%20applied%20nonlinear%20control-%20bad%20quality.pdf)

- **[Haddad and Chellaboina, 2008. Nonlinear Dynamical Systems and Control: A Lyapunov-Based Approach](https://www.jstor.org/stable/j.ctvcm4hws)**

#### Papers (Chronological):

- [Artstein, 1982. Stabilization with Relaxed Controls](https://www.sciencedirect.com/science/article/pii/0362546X83900494)

- [Sontag, 1983. A Lyapunov-Like Characterization of Asymptotic Controllability](https://epubs.siam.org/doi/abs/10.1137/0321028?casa_token=UBCu6QQcDNQAAAAA:gnQ5sEIIyS-uXT8bzuob6UIffSQjAbhfVmpw8qx_cxOJ-RmMJT25gE1Hb5iwxCJqXlYsIl9J)

- [Sontag, 1989. A 'Universal' Construction of Artstein's Theorem on Nonlinear Stabilization](http://www.sontaglab.org/FTPDIR/art-sycon8903.pdf)

- **[Ames et al., 2016. Control Barrier Function Based Quadratic Programs for Safety Critical Systems](https://ieeexplore.ieee.org/abstract/document/7782377?casa_token=Su26OE1ZV1EAAAAA:KMlvhbhQq0wb76ixit1QMQgIVqZEQj8Vf4fTlfewe5_auWi48Zb-nSSJ7mxopBuDgF0bLT3BHw)**

- [Ames et al., 2019. Control Barrier Functions: Theory and Applications](https://coogan.ece.gatech.edu/papers/pdf/amesecc19.pdf)

- [Capelli et al., 2020. Connectivity Maintenance: Global and Optimized Approach Through Control Barrier Functions](https://ieeexplore.ieee.org/abstract/document/9197109?casa_token=HEt2jergrKEAAAAA:oStQJxHycKbc4iCHloCpSi62P8oWoPqILA4k_tcUEYNrMS76EK40c4lyLcJSjdBnA5LlL7E)

- [Cavorsi et al., 2023. Multirobot Adversarial Resilience Using Control Barrier Functions](https://ieeexplore.ieee.org/abstract/document/10354416?casa_token=6jyiwNV7sCEAAAAA:5BLuAUeRw1ZmuZaxHnD_YfWKkL0wZqbVP8pHQNU8xamrJAbb9cJMJGvkxzNIYrNsIZ59Or0)

<!---
https://www.ll.mit.edu/partner-us/available-technologies/control-barrier-function-toolbox
-->
