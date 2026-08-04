---
title: "Hybridization of Neural Networks and Numerical Solvers in JAX with Differentiable Physics (Lecture and Practical Workshop)"
date: 2026-08-04
permalink: /posts/2026/08/jax-ad-workshop/
author_profile: true
tags:
    - PDE
---

{% include base_path %}

I was invited to give a lecture and practical sessions at the [Workshop on
Machine Learning and Automatic Differentiation in `JAX` for Scientific Computing
](https://majsc2026.pages.math.unistra.fr/) hosted by the University of
Strasbourg in June 2026. For this, I presented a lecture on hybridization (i.e.,
how to marry neural networks and numerical solvers) as well as a practical
session on using the [Exponax](https://fkoehler.site/exponax/) solver suite in
JAX for solving PDEs (concretely the Kolmogorov Flow), data assimilation,
inverse problems, and to build neural-hybrid correctors leading to a simplified
reproduction of the influential [ML-accelerated CFD paper by Kochkov et al.](https://www.pnas.org/doi/10.1073/pnas.2101784118). Thank you for having me!

All material can be found on [this GitHub repository](https://github.com/Ceyron/hybridization-in-jax).

Recording of the lecture:

[![Link to the lecture recording](https://img.youtube.com/vi/carwzAOfuPE/0.jpg)](https://www.youtube.com/watch?v=carwzAOfuPE)

Recording of the "zeroth" practical session on getting started with Exponax by solving the 2D Navier-Stokes Kolmogorov Flow:

[![Link to the "zeroth" practical session recording](https://img.youtube.com/vi/I7ilrh6tbVc/0.jpg)](https://www.youtube.com/watch?v=I7ilrh6tbVc)

Recording of the first practical session on data assimilation and inverse problems:

[![Link to the first practical session recording](https://img.youtube.com/vi/Tb3n52Ka4GM/0.jpg)](https://www.youtube.com/watch?v=Tb3n52Ka4GM)

Recording of the second practical session on building neural-hybrid correctors:

[![Link to the second practical session recording](https://img.youtube.com/vi/g1guv-fkIrQ/0.jpg)](https://www.youtube.com/watch?v=g1guv-fkIrQ)

Below is the abstract of the talk:

> Neural Networks and Numerical Solvers for PDEs share strong similarities. Both
> boil down to algebra operations that are mapped to primitives on hardware
> accelerators. In this lecture, we will compare both from an algebraic and an
> autodiff perspective, identifying advantages and weak points of both. After
> discussing a range of possible ways neural networks can be used to approximate
> physics simulations and how the two concepts can be married, we will settle on
> full-field correction approaches as the most "deep-learning native way" of
> hybridization. We will discuss nuances, technical insights, and practical JAX
> implementations. The talk closes with an overview of case studies from the
> literature.

Below is the abstract of the practical sessions:

> We will use the Exponax Fourier-spectral solver library
> (https://github.com/Ceyron/exponax ) to perform fast simulation of the
> Kolmogorov Flow, a well-studied test bed of 2D turbulence. By using the
> solver's differentiability, we can recover initial conditions and constitutive
> parameters from data. The second half of this practical is concerned with
> reproducing a simplified scenario of the influential "Machine
> learning-accelerated computational fluid dynamics" paper by Kochkov et al.
> (https://www.pnas.org/doi/10.1073/pnas.2101784118 ). This entails learning a
> neural correction approach for a simulation on a coarser resolution. The
> latter part of the exercise requires access to a GPU. The free T4 GPU from
> Google Colab (e.g., conveniently accessible via the VS Code plugin:
> https://github.com/googlecolab/colab-vscode ) is sufficient.
