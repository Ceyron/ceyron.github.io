---
title: "A general strategy for derivaing autodiff rules (for PDEs)"
date: 2023-03-12
permalink: /posts/2023/03/autodiff-rule-derivation/
tags:
    - autodiff
    - jvp
    - vjp
    - differentiation
---

**⚠️ ⚠️ ⚠️ This blog post is still under heavy construction.**

Consider a general description of a stationary PDE problem with homogeneous Dirichlet boundary conditions:

$$
\begin{cases}
\mathcal{L}\left(u(x), \theta\right) = 0, \quad & x \in \Omega, \\
u(x) = 0, \quad & x \in \partial\Omega,
\end{cases}
$$

This problem is in terms of a differential operator $\mathcal{L}$, a parameter vector $\theta$, and a domain $\Omega$. We could, for instance have an advection-diffusion equation with a parameter- and space-dependent driving velocity $c(c, \theta)$:

$$
\mathcal{L}\left(u(x), \theta\right) = c(x, \mathbf{\theta}) \frac{\partial u}{\partial x} + \nu \frac{\partial^2 u}{\partial x^2}
$$

Let's say we have access to an abstract PDE integrator, which given the parameter vector $\mathbf{\theta}$ can solve for the solution function $u(x)$

$$
q(\mathbf{\theta}) =
\left \{
    \text{integrate} \; \mathcal{L}(u(x), \mathbf{\theta}) = 0 \; \text{for} \; u(x) \; \text{with} \; u|_{x \in \partial \Omega} = 0
\right \}
$$

For automatic differentiation we are now interested in the primitives for forward- and reverse-mode. Having a rule on the level of PDE integration helps us avoid unrolling and differentiating **through** the PDE solver. These rules are:

* Pushforward/Jvp: $(\theta, \dot{\theta}) \mapsto (u, \dot{u}) = (q(\theta), \langle \frac{\partial q}{\partial \theta}, \dot{\theta} \rangle)$
* Pullback/vJp: $(\theta, \bar{u}) \mapsto (u, \bar{\theta}) = (q(\theta), \left( \frac{\partial q}{\partial \theta} \right)^T \bar{u}$

## Forward-Mode: Pushforward Rule

The forward-mode is concerned with the inner product of the Jacobian of the integration wrapper with the tangent on the input

$$
\dot{u} = \langle \frac{\partial q}{\partial \theta}, \dot{\theta} \rangle
$$

Implicitly deriving the PDE gives

$$
\frac{d}{d\theta}\mathcal{L}
=
\frac{\partial \mathcal{L}}{\partial u}
\frac{\partial u}{\partial \theta} + \frac{\partial \mathcal{L}}{\partial \theta} = 0
$$

Taking the inner product with the parameter tangent
$$
\langle \frac{\partial \mathcal{L}}{\partial \theta} \frac{\partial u}{\partial \theta}, \dot{\theta} \rangle + \langle \frac{\partial \mathcal{L}}{\partial \theta}, \dot{\theta} \rangle = 0
$$

We can extract the linearized differential operator from the left inner product to get

$$
\frac{\partial \mathcal{L}}{\partial \theta} \underbrace{\langle \frac{\partial u}{\partial \theta}, \dot{\theta} \rangle}_{=\dot{u}} = - \langle \frac{\partial \mathcal{L}}{\partial \theta}, \dot{\theta} \rangle
$$

which gives rise to the tangent linear auxiliary PDE problem in terms of the solution tangent $\dot{u}$.

$$
\dot{u}
=
\left \{
    \text{integrate} \; \frac{\partial \mathcal{L}}{\partial \theta} \dot{u} = - \langle \frac{\partial \mathcal{L}}{\partial \theta}, \dot{\theta} \rangle
    \; \text{for} \; \dot{u}(x) \; \text{with} \; \dot{u}|_{x \in \partial \Omega} = 0
\right \}
$$

## Reverse-Mode: Pullback Rule

Consider again the total derivative of the implicit PDE problem

$$
\frac{d}{d\theta}\mathcal{L}
=
\frac{\partial \mathcal{L}}{\partial u}
\frac{\partial u}{\partial \theta} + \frac{\partial \mathcal{L}}{\partial \theta} = 0
$$

Take the inner product with an adjoint variable

$$
\langle \lambda, \frac{\partial \mathcal{L}}{\partial \theta} \frac{\partial u}{\partial \theta}\rangle + \langle \lambda, \frac{\partial \mathcal{L}}{\partial \theta} \rangle = 0
$$

Perform integration by parts on the left inner product. Note that we have homogeneous Dirichlet boundary conditions.

$$
- \langle \left( \frac{\partial \mathcal{L}}{\partial \theta} \right)^T  \lambda, \frac{\partial u}{\partial \theta}\rangle + \langle \lambda, \frac{\partial \mathcal{L}}{\partial \theta} \rangle = 0
$$

Add an artificial zero

$$
- \langle \left( \frac{\partial \mathcal{L}}{\partial \theta} \right)^T  \lambda, \frac{\partial u}{\partial \theta}\rangle + \langle \lambda, \frac{\partial \mathcal{L}}{\partial \theta} \rangle = 
\langle \bar{u}, \frac{\partial u}{\partial \theta} \rangle
- \langle \bar{u}, \frac{\partial u}{\partial \theta} \rangle
$$

We can identify an adjoint PDE problem in terms of the adjoint variable

$$
\lambda(x) = \left \{
    \text{integrate} \; \left( \frac{\partial \mathcal{L}}{\partial \theta} \right)^T \lambda = \bar{u}
    \; \text{for} \; \lambda(x) \; \text{with} \; \lambda|_{x \in \partial \Omega} = 0
\right \}
$$

Finally, we pullback with the inner product

$$
\langle \bar{u}, \frac{\partial u}{\partial \theta} \rangle
= - \langle \lambda, \frac{\partial u}{\partial \theta} \rangle
$$