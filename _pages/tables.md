---
layout: archive
title: "Overview Tables"
permalink: /tables/
author_profile: true
redirect_from:
  - /overview-tables
  - /overviews
---

{% include base_path %}

## Tables on Automatic Differentiation

* [Overview of Explicit Automatic Differentiation Rules](/autodiff-table) for both scalar and tensor operations (forward- and reverse-mode)
* [Overview of Implicit Automatic Differentiation Rules](/implicit-autodiff-table) like root-finding, (non-)linear system solving, ODE and PDE integration or differentiable optimization (forward- and reverse-mode)

Below tables are experimental and not yet complete.
* [Overview of 1D Convolution Automatic Differentiation
  Rules](/conv-autodiff-table) with different padding and stride options (only
  reverse-mode, the forward-mode is straightforward based on the matrix-vector
  multiplication rule)
* [Overview of Convolution Automatic Differentiation Rules in various deep
  learning frameworks](/conv-autodiff-table-frameworks) (only reverse-mode, the
  forward-mode is straightforward based on the matrix-vector multiplication
  rule): this differs from the above table in that:
  1. some frameworks (like PyTorch, TensorFlow or JAX) use cross-correlation
     instead of convolution
  2. real-world convolutions/cross-correlations often map multiple channels to
     multiple channels and do so for multiple samples (one batch) at once (that
     requires additional reordering of tensor axes for the reverse passes)
  3. the tensor layout convention (of batch, spatial and channel axes) can vary
     between frameworks

## Tables on Physics-based deep learning

* [Overview of learning setups for neural predictors](/predictor-learning-setups) like the classical t-step supervised, but also more that involve differentiable physics simulators
* [Overview of corrector configurations](/corrector-configurations) for neural correctors that are trained to augment coarse solvers
