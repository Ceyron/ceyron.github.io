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

## Tables on Fast Fourier Transform (FFT)

* [Relation of functions and their Fourier coefficients in 1d](/fourier-table):
  shows, for instance
  1. how the coefficients are scaled
  2. that a real positive cosine has both positive real coefficients at positive **and** negative
    wavenumbers
  3. that a real positive sine has a negative imaginary coefficient at the positive wavenumber and
    a positive imaginary coefficient at the negative wavenumber
  4. how imaginary sines and cosines are represented
  5. how the Nyquist mode glitches at an even number of sampling points
  6. how higher modes are aliased
* [Similar table for 2d](/fourier-table-2d): here, we additionally have:
  1. how the indexing scheme for `np.meshgrid` affects the order of the coefficients
  2. how the scaling of the coefficients change (N scaling for zero mode, N/2
     scaling for modes where one axis is the zero mode, N/4 scaling for all
     other modes (except the Nyquist mode at even numbers of sampling points))
  3. how the Nyquist mode glitches, especially if one axis has an even number of
     sampling points and the other an odd number
