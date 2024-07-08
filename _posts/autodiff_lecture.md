---
permalink: /autodiff_lecture/
title: "Lecture: Automatic Differentiation & Adjoint Methods in Differentiable Physics"
author_profile: true
tags:
    - autodiff
    - adjoint
    - differentiable physics
    - PDE
---

{% include base_path %}

As part of our master course in [Advanced Deep Learning for Physics (IN2298)](https://ge.in.tum.de/teaching/), I gave a lecture on autodiff and adjoint methods. You can find the lecture slides [here](https://fkoehler.site/files/autodiff_and_adjoints_lecture.pdf). The lecture was recorded and is available on YouTube:

[![Link to the YouTube video](https://img.youtube.com/vi/N7nVoyR0qO4/0.jpg)](https://www.youtube.com/watch?v=N7nVoyR0qO4)

In it, I cover:

1. A functional (JAX/Julia-inspired) viewpoint on autodiff in terms of Jvp/Pushforward and vJp/Pullback
2. Identifying hierarchy levels in autodiff (scalar-mode, vector-mode, continuous-mode)
3. A comparison of Optimize-then-Discretize (OtD) vs. Discretize-then-Optimize (DtO)
4. Special aspects of differentiable physics like differentiating over linear and nonlinear solvers
5. Advanced topics and recent research directions