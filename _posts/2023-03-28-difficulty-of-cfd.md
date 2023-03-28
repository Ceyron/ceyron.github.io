---
title: "Why are fluid flow so difficult to simulate numerically?"
date: 2023-03-28
permalink: /posts/2023/03/difficulty-of-cfd/
tags:
    - cfd
    - numerics
    - simulation
    - pde
---

**⚠️ ⚠️ ⚠️ This blog post is still under heavy construction.**

The Navier-Stokes equation fascinated me since the very first time I heard about
it back in my undergraduate course on fluid mechanics. Back then, it was quite
frightening: the partial derivatives $\partial u / \partial t$, the nabla
operator $\nabla$ and just so many operators all advance. I was intriguied. With
the numerics courses that preceeded this fluid mechanics introduction I came
closer and closer to building an understanding of how one would solve such an
equation. However, it was not until my master thesis that I had to start delving
into the details and difficulties of simulation techniques: time to reflect and
answer the question: why is the Navier-Stokes equation so difficult to solve and
how do people actually solve it?

First of all, when I talk about the Navier-Stokes equation, I actually refer to
its incompressible version. This is a set of partial differential equations in
two dimensions ($=\Re^2$) or three dimensions ($=\Re^3$). The two or three
equations for the momentum of the fluid are subject to an incompressibility
constraint (a third or fourth equation). In a convenient vector notation, one
would write

$$
\underbrace{
    \frac{\partial u}{\partial t}
}_{\text{transient}}
+
\underbrace{
    (u \cdot \nabla) u
}_{\text{advection}}
=
\underbrace{
    -\nabla p 
}_{\text{pressure gradient}}
+
\underbrace{
    \frac{1}{Re}
    \nabla^2u
}_{\text{diffusion}}
+
\underbrace{
    f
}_{\text{external force}}
\\
\underbrace{
    \nabla \cdot u = 0
}_{\text{incompressibility / continuity}}
$$

The pressure gradient can be thought of a way to enforce the incompressibility.
It acts as a Lagrange multiplier. Notice also, that we deal with the
dimensionless-version of the Navier-Stokes equation that is solely characterized
by the Reynolds number. A high Reynolds number means that the coefficient in
front of the diffusion becomes small. As such, there will be little diffusion
and more advection. Vice versa, a high Reynolds number refers to a more
diffusive flow.

## Why is the simulation difficult?

Essentially, the Navier-Stokes equation is a finale in the combination of
modeling building blocks one has for second order partial differential equation.
It is transient, has a hyperbolic advection part, a parabolic diffusion part,
can be driven a force/reaction term and it requires the flow to be
incompressible. Similar to learning about PDEs and their simulation, diffusion
is the easiest component of the Navier-Stokes equation. Indeed, strong diffusion
is often helpful as it stabilizes weaker numerical schemes. On the other hand,
most real-world flows have high Reynolds numbers (typically $\gg10^5$) that
refer to low-diffusion flows. There are four major challenges one has to overcome:

1. The strong dominance of advection requires upwind-biased discretization schemes.
2. The nonlinearity in the advection.
3. The saddle-point structure due to the incompressibility constraint.
4. The difficulty of capturing all scales in turbulence.

They are ordered by the difficulty they induce. Let's tackle them one-by-one.

### The difficulty with advection-dominated flows

The model problem for a hyperbolic PDE is the 1D advection equation

$$
\frac{\partial u}{\partial t}
+
c \frac{\partial u}{\partial x}
=
0
$$

If we applied a forward-in-time central-in-space scheme to it

$$
u_i^{[t+1]}
\leftarrow
u_i^{[i]}
-
c
\Delta t
\frac{u_{i+1}^{[t]} - u_{i-1}^{[t]}}{2 \Delta x}
$$

one can show by von-Neumann stability analysis that this scheme is
**unconditionally unstable**. In other words: There is no time step size $\Delta
t$ for which this scheme can be stable; it will always develop oscillations.

The solution is to use upwind-biased schemes or specialized advection schemes. A
protoptyical solution would be a first-order upwind scheme

$$
u_i^{[t+1]}
\leftarrow
u_i^{[i]}
-
c
\Delta t
\begin{cases}
\frac{u_{i}^{[t]} - u_{i-1}^{[t]}}{\Delta x}
\quad c \ge 0
\\
\frac{u_{i+1}^{[t]} - u_{i}^{[t]}}{\Delta x}
\quad c < 0
\end{cases}
$$

which is **conditionally stable** according to the CFL condition

$$
\Delta t \leq \frac{\Delta x}{c}
$$

These upwind-biased schemes make matrix assemblies difficult. Additionally, if
one wants to use higher-order methods, one suffers from [Godunov's
theorem](https://en.wikipedia.org/wiki/Godunov%27s_theorem). As such, a method
of order greater than one (hence, everything that is not firsr-order upwind)
will develop oscillations around steep discontinuities.
