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

### The problem with nonlinearity

Since then fluid *advects itself*, the transport term is nonlinear. The
protopyical example for such a term is the 1D Burgers equation

$$
\frac{\partial u}{\partial t}
+
u \frac{\partial u}{\partial x}
=
0
$$

Physically, this equation is prone to build shocks and rarefaction waves.
However, those are typically not relevant for incompressible Navier-Stokes
simulations due to the additional continuity constraint. More interesting to us
is how to solve this nonlinear equation.

If we wanted to integrate our Navier-Stokes equation with the transport term
treated explicitly, the nonlinearity would be convieniently caputered by the
time stepping scheme. For example with first-order upwind.

$$
u_i^{[t+1]}
\leftarrow
u_i^{[i]}
-
u_i^{[t]}
\Delta t
\begin{cases}
\frac{u_{i}^{[t]} - u_{i-1}^{[t]}}{\Delta x}
\quad u_i^{[t]} \ge 0
\\
\frac{u_{i+1}^{[t]} - u_{i}^{[t]}}{\Delta x}
\quad u_i^{[t]} < 0
\end{cases}
$$

Note that since the 'velocity' of the advection equation is the unknown itself,
it also affects the wind direction and hence the direction of discretization.
Also, the CFL condition now becomes time-level dependent

$$
\Delta t \leq \frac{\Delta x}{\max(|u|)}
$$

Obviously, for fast flows (often related to high Reynolds numbers) we need small
time steps.

As auch, one is more often interested in **implicit** discretizations in which
the solution at the next time step is found as the solution to a (nonlinear)
system of equations. There are two major approaches:

1. Picard iteration (including the Picard1 method that just leads to one linear
   system solve per time-step, but introduces a $\mathcal{O}(\Delta t)$ error)
2. Newton-Raphson iteration

Both are established techniques in scientific computing, but require a lot of
fine-tuning to work efficiently. One particular difficulty is the solution to
large sparse linear systems of equations which often are nonlinear for
advection-dominated problems.

### The difficulty of the saddle point structure

We can also write down the structure of the incompressible Navier-Stokes
equation as

$$
\begin{bmatrix}
\frac{\partial}{\partial t} + (u \cdot \nabla) - \frac{1}{Re} \nabla^2 & - \nabla
\\
\nabla \cdot & 0
\end{bmatrix}
\begin{bmatrix}
u
\\
p
\end{bmatrix}
=
\begin{bmatrix}
f
\\
0
\end{bmatrix}
$$

The two or three momentum equations are represented in the first row of the
matrix whereas the incompressibility constrain is in the second. This structure
of problem, with a zero block in the bottom right is called a saddle-point
problem. It typically arises when using Lagrange multipliers to solves
constrained equations.

The arising challenge is due to the extremely high condition number of the
problem. As such, linear solvers applied to this problem will converge very
slowly. Using effective preconditioners, therefore, is a must. Many popular
algorithms like *SIMPLE*, *PISO* or *COUPLED* arise as special forms for
preconditioners found under certain assumptions.

Still, the effective formulation of preconditioners is problem-dependent.

### The challenge of multiple scales

The nonlinearity (and the incompressibility constraint) in the Navier-Stokes
equation gives rise to turbulence, a chaotic behavior of the fluid. Upon a
certain Reynolds number, a flow transitions from a laminar state into a
turbulent state. It becomes more and more turbulent the higher Reynolds number
is. Since realistic Reynolds numbers are quite high (usually $\gg 10^5$), almost
all practically relevant flows are turbulent. From the works of Kolmogorov, we
know that the more turbulent a flow is (i.e., the higher the Reynolds number),
the finer structure it develops. In order to capture these structures, one needs
ever finer meshers and smaller time steps. If we did not capture these
structures, our simulation would be incorrect. [One can
show](https://www.youtube.com/watch?app=desktop&v=Depe06jFNis) shat the workload
(e.g., in terms of the number of floating point operations) scales as
$\mathcal{O}(Re^3)$ rendering the direct numerical simulation of realistic flows
infeasible on even the fastest supercomputers.

A remedy is the clever usage of a multiscale model, a turbulence model. This is
a science for itself. There are many well-established robust models, that are
physically grounded. However, the exact usage of them is highly
problem-dependent and as every model they are imperfect. If their accuracy
matches the acceptable tolerance of the simulation, they will work sufficiently.


## The structure of (almost) all Navier-Stokes solvers

I really liked the book [Efficient Solvers for Incompressible Flow
Problems](https://link.springer.com/book/10.1007/978-3-642-58393-3) by Stefan
Turek. It tries to unify the majority of available algorithms into a
'Navier-Stokes tree'

![image](https://user-images.githubusercontent.com/27728103/228235324-b561df29-00ea-4c74-9f6a-eb0704ba1734.png)

Let's go through each level

### 1. The fully continuous level

The Navier-Stokes equation is given with the nonlinear advection and the diffusion packed into a
nonlinear differential operator $N(u)$ to get

$$
u_t
+
N(u)u
+
\nabla p
=
f
,
\quad
\nabla \div u = 0
\quad
\text{in 2D/3D}
+
\text{B.C.}
$$

We can write the differential operator as

$$
N(u) := (u \cdot \nabla) - \frac{1}{Re} \nabla^2 
$$

### 2. The time-discrete level

We want to use implicit time stepping mechanisms to be (theoretically) able to
use any time step size and still be stable. The flow scheme indicates a backward
eueler (BE) and a Crank-Nicolson (CN) technique. Any influence from a solution at a
previous time level is absorbed in the new right-hand side $g$. Instead of an
initial-boundary value problem, we now only have a boundary value problem in
terms of the solution state at the next time step which reads

$$
\left[
    I
    +
    \theta k N(u)
\right]
u
+
k \nabla p
=
g
\quad
\nabla \cdot u = 0
$$

### 3. The fully discrete level

One can now apply a spatial discretization method of choice to translate the
boundary-value problem into a fully discrete set of equations

$$
\left[
    M
    +
    \theta
    k
    N(u_h)
\right]u_h
+
k B p_h
=
g_h
\quad
B^Tu_h =0
$$

together with the mass matrix $M$ and the gradient matrix $B$ whose transpose
$B^T$ is the divergence matrix. This method is chosen such that the advection is
treated stabily.

By merging the square backed into one nonlinear function

$$
S(u_h)
=
M
+
\theta
k
N(u_h)
$$

we can write the coupled system as 

$$
\begin{aligned}
S(u_h)u_h + kBp_h &= g_h
\\
B^Tu_h &= 0
\end{aligned}
$$

### 4. The level of the nonlinear saddle-point solver

The coupled system is

1. nonlinear, because it's system matrix $S$ is not constant, but a function of
   the unknown $S = S(u_h)$
2. a saddle-point problem, because it has a zero block in the bottom right

There is no direct solution to either of the problems. Hence, we have to iterate
in order to find the solution to the problem. Naturally, we have two ways to
nest the iterations:

1. Treat the nonlinearity in the outer iteration and the saddle point in the
   inner iteration, which here is referred to as **Galerkin schemes**
2. Decouple the two equations and thereby solver the saddle point problem in an
   outer iteration and the nonlinearity in the inner iteration, which here is referred to as **Projection schemes**

### 4.1 Galerkin schemes

If we solve the nonlinear system of equations in terms of a Picard iteration,
the problem turns into

$$
\begin{aligned}
S(u_h^{[k]})u_h^{[k+1]} + kBp_h^{[k+1]} &= g_h
\\
B^Tu_h^{[k+1]} &= 0
\end{aligned}
$$

For performing one iteration, the system matrix is linearized around the
previous iterate $S(u_h^{[k]})$. Hence, the system will be just a linear system
of equation. It is also referred to as an Oseen problem. We have to repeat this
solution to the linear Oseen problem multiple times (the number of iterates is
denoted by $L$). There are two major approaches:

### 4.1.1 Exact Schur Complement

Only requires one iteration to solve the Osee problem

In the notation of the book this is called the *CC2D* and *CC3D* for the
two-dimensional and the three-dimensional case, respectively.

### 4.1.2 Approximate Schur Complement

Requires multiple steps to solve the saddle-point problem

The book calls it the *CP2D* and *CP3D*

### 4.2 Projection schemes

They differentiate between wether to do one or multiple projection solves

### 4.2.1 One Projection Solve

This includes the methods due to Chorin and Van Kan

We can now differentiate wether we do a Picard1 iteration of the Burgers
equation (which is the sub-problem or apply an error control)

### 4.2.1.1 One Projection Solve with one nonlinear iteration

This introduces a $\mathcal{O}(\Delta t)$ error.

### 4.2.1.2 One Projection Solve and converged nonlinear solve

### 4.2.2 Multiple Projection Solves

This includes the famous *SIMPLE*, *PISO* and *SIMPLEC* schemes.
