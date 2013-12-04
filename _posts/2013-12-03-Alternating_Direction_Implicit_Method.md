---
layout: default
title: "The Alternating Direction Implicit Method"
date: 2013-12-03
tags: wip
---

**This post is work in progress**

# The Alternating Direction Implicit Method

Just as the [Crank-Nicolson (CN)
method](http://georg.io/2013/12/03/Crank_Nicolson.html) for reaction-diffusion
systems with one space dimension, the
[Alternating Direction Implicit (ADI)
method](http://en.wikipedia.org/wiki/Alternating_direction_implicit_method)
is used commonly for reaction-diffusion systems with two space dimensions.

The ADI method has been described thoroughly many times, for instance by
[Dehghan](http://www.sciencedirect.com/science/article/pii/S0377042700004520).

In the following discussion we will consider a reaction-diffusion system similar
to the one we
studied [previously](http://georg.io/2013/12/03/Crank_Nicolson.html) and we will
use analogous
[Neumann boundary
conditions](http://en.wikipedia.org/wiki/Neumann_boundary_condition).

$$\frac{\partial u}{\partial t} = D \left( \frac{\partial^2 u}{\partial x^2} +
\frac{\partial^2 u}{\partial y^2} \right) + f(u),$$

$$\frac{\partial u}{\partial x}\Bigg|_{x = 0, L_x} = 0,$$

$$\frac{\partial u}{\partial y}\Bigg|_{y = 0, L_y} = 0,$$

where $u(x,y,t)$ is our concentration variable, $D$ is the diffusion coefficient
of $u$, $f$ is the reaction term, and $L_x$ and $L_y$ are the extent of our
domain in the $x$ and $y$ direction respectively.

Since ADI and CN are somewhat related, let us try to derive the most basic
properties of the ADI in relation to CN.

## Grid Construction

Analogous to the $(x,t)$-grid
[used in
CN](http://georg.io/2013/12/03/Crank_Nicolson.html#finite_difference_methods),
we need to construct an $(x,y,t)$-grid for this problem with two space
dimensions.

In the simplest case, we construct a [regular
grid](http://en.wikipedia.org/wiki/Regular_grid) as follows

$$t_n = n \Delta t,~ n = 0, \ldots, N-1,$$

$$x_j = j \Delta x,~ j = 0, \ldots, J-1,$$

$$y_i = i \Delta y,~ i = 0, \ldots, I-1,$$

where $N$, $J$, and $I$ are the number of grid points in the $t$-, $x$-, and
$y$-direction.

$\Delta t$, $\Delta x$, and $\Delta y$ are defined as follows

$$\Delta t = \frac{T}{N-1},~ \Delta x = \frac{L_x}{J-1},~ \Delta y =
\frac{L_y}{I-1},$$

where $T$ is the total amount of time we are interested in.

[As before]((http://georg.io/2013/12/03/Crank_Nicolson.html#finite_difference_me
thods) we will refer to the
numerical approximations of the unknown analytic solution $u(x,y,t)$ in our grid
points as

$$U(j \Delta x, i \Delta y, n \Delta t) \approx u(j \Delta x, i \Delta y, n
\Delta t),$$

and we use the shorthand $U(j \Delta x, i \Delta y, n \Delta t) = U_{j,i}^n.$
We also refer to the grid point $(j \Delta x, i \Delta y, n \Delta t)$ as
$(j,i,n)$.

## Motivation for ADI

When integrating the above reaction-diffusion equation numerically, we can still
make use of the
[CN stencil](http://georg.io/2013/12/03/Crank_Nicolson.html#the_cranknicolson_st
encil).

Applying the CN stencil to our reaction-diffusion equation on our
$(x,y,t)$-grid, we obtain:

$$\frac{U_{j,i}^{n+1} - U_{j,i}^n}{\Delta t} =
\frac{D}{2 \Delta x^2} \left( U_{j+1,i}^n -
2 U_{j,i}^n + U_{j-1,i}^n + U_{j+1,i}^{n+1} - 2 U_{j,i}^{n+1} +
U_{j-1,i}^{n+1}\right) +
\frac{D}{2 \Delta y^2} \left( U_{j,i+1}^n -
2 U_{j,i}^n + U_{j,i-1}^n + U_{j,i+1}^{n+1} - 2 U_{j,i}^{n+1} +
U_{j,i-1}^{n+1}\right) +
f(U_{j,i}^n).$$

Let us define $\sigma_x = \frac{D \Delta t}{2 \Delta x^2}$ and $\sigma_y =
\frac{D \Delta t}{2 \Delta y^2}$.

To [reorder our stencil](http://georg.io/2013/12/03/Crank_Nicolson.html#reorderi
ng_stencil_into_linear_system) into a linear system
we need to define a new index that combines indices $i$ and $j$:

$$k = j + i J$$

This new index $k$ flattens the two-dimensional spatial part of our
$(x,y,t)$-grid into a (one-dimensional) vector
and our "flattening methodology" is analogous to the [row-major
order](http://en.wikipedia.org/wiki/Row-major_order)
representation of matrices.

To illustrate, when we use the $k$ index grid points $(j,i,n)$ and $(j,i+1,n)$
become $(k,n)$ and $(k+J,n)$ respectively.

Our stencil therefore becomes

$$U_{k}^{n+1} - U_{k}^n =
\sigma_x \left( U_{k+1}^n -
2 U_{k}^n + U_{k-1}^n + U_{k+1}^{n+1} - 2 U_{k}^{n+1} + U_{k-1}^{n+1}\right) +
\sigma_y \left( U_{k+J}^n -
2 U_{k}^n + U_{k-J}^n + U_{k+J}^{n+1} - 2 U_{k}^{n+1} + U_{k-J}^{n+1}\right) +
\Delta t f(U_{k}^n),$$

and reordering this expression, we obtain

$$
-\sigma_y U_{k-J}^{n+1}
-\sigma_x U_{k-1}^{n+1}
+ (1 + 2 \sigma_x + 2 \sigma_y) U_{k}^{n+1}
-\sigma_x U_{k+1}^{n+1}
-\sigma_y U_{k+J}^{n+1} =
\sigma_y U_{k-J}^n
+\sigma_x U_{k-1}^n
+(1 - 2 \sigma_x - 2 \sigma_y) U_{k}^n
+\sigma_x U_{k+1}^n
+\sigma_y U_{k+J}^n
+\Delta t f(U_{k}^n).$$

If we [wrote this out in matrix notation](http://georg.io/2013/12/03/Crank_Nicol
son.html#reordering_stencil_into_linear_system), we would
see
[banded matrices](http://publib.boulder.ibm.com/infocenter/clresctr/vxrx/index.j
sp?topic=%2Fcom.ibm.cluster.essl.v5r2.essl100.doc%2Fam5gr_bandma.htm)
both on the left- and right-hand side (analogous to matrices $A$ and $B$
[in the one-dimensional case](http://georg.io/2013/12/03/Crank_Nicolson.html#reo
rdering_stencil_into_linear_system)):

On the left-hand side of this liner system, matrix $A$ contains a
tridiagonal core [as before](http://georg.io/2013/12/03/Crank_Nicolson.html#reor
dering_stencil_into_linear_system) with
non-zero elements, $-\sigma_y$, in the $J$-th [sub- and
superdiagonals](http://en.wikipedia.org/wiki/Diagonal#Matrices).
Matrix $B$ on the right-hand side looks similar but with $\sigma_y$ in the
$J$-th sub- and superdiagonals.

Numerical inversion of banded matrices, such as $A$, is
[expensive](http://en.wikipedia.org/wiki/Alternating_direction_implicit_method)
with standard methods - at the very least we would not be able to use the
standard
[Thomas algorithm](http://en.wikipedia.org/wiki/Tridiagonal_matrix_algorithm)
for tridiagonal matrices.

The [ADI
stencil](http://en.wikipedia.org/wiki/Alternating_direction_implicit_method)
makes use of a trick that allows us to invert
tridiagonal matrices, just as with the
[CN stencil](http://georg.io/2013/12/03/Crank_Nicolson.html#the_cranknicolson_st
encil),
instead of banded matrices.

The ADI stencil also suggests a straightforward way to implement a parallelized
variant of the ADI method and we will discuss and
demonstrate this below.

## The ADI Stencils

The trick used in constructing the ADI method is to split our time step $\Delta
t$ into two and apply two different stencils in each
half time step:
therefore to increment time by one time step $n \rightarrow n+1$ in grid point
$(j,i,n)$, we first compute $U_{j,i}^{n+1/2}$
($n \rightarrow n+1/2$) and then compute $U_{j,i}^{n+1}$ ($n+1/2 \rightarrow
n+1$).
Both of these stencils are chosen such that the resulting linear system is
tridiagonal.

The two [ADI
stencils](http://en.wikipedia.org/wiki/Alternating_direction_implicit_method)
bring in the x- and y-direction at the next time point
([implicit](http://en.wikipedia.org/wiki/Finite_difference_method#Implicit_metho
d) v
[explicit](http://en.wikipedia.org/wiki/Finite_difference_method#Explicit_method
) finite difference stencils) alternatingely -
inspiring the name of the ADI method:

$$U_{j,i}^{n+1/2} - U_{j,i}^n =
\sigma_x \left( U_{j+1,i}^{n+1/2} - 2 U_{j,i}^{n+1/2} + U_{j-1,i}^{n+1/2}
\right) +
\sigma_y \left( U_{j,i+1}^n - 2 U_{j,i}^n + U_{j,i-1}^n \right) +
\Delta t f(U_{j,i}^n),$$

$$U_{j,i}^{n+1} - U_{j,i}^{n+1/2} =
\sigma_x \left( U_{j+1,i}^{n+1/2} - 2 U_{j,i}^{n+1/2} + U_{j-1,i}^{n+1/2}
\right) +
\sigma_y \left( U_{j,i+1}^{n+1} - 2 U_{j,i}^{n+1} + U_{j,i-1}^{n+1} \right) +
\Delta t f(U_{j,i}^{n+1/2}).$$

## The ADI Linear Systems

Reordering both stencils, we obtain the following systems of linear equations

$$
-\sigma_x U_{j-1,i}^{n+1/2} + (1+2\sigma_x) U_{j,i}^{n+1/2} - \sigma_x
U_{j+1,i}^{n+1/2} =
\sigma_y U_{j,i-1}^n + (1-2\sigma_y) U_{j,i}^n + \sigma_y U_{j,i+1}^n + \Delta t
f(U_{j,i}^n)
$$

$$
-\sigma_y U_{j,i-1}^{n+1} + (1+2\sigma_y) U_{j,i}^{n+1} - \sigma_y
U_{j,i+1}^{n+1} =
\sigma_x U_{j-1,i}^{n+1/2} + (1-2\sigma_x) U_{j,i}^{n+1/2} + \sigma_y
U_{j+1,i}^{n+1/2} + \Delta t f(U_{j,i}^{n+1/2})
$$

### Linear System in the x-Direction

Stencil pictured in [Dehghan Figure
1](http://www.sciencedirect.com/science/article/pii/S0377042700004520#FIG1)

### Linear System in the y-Direction

## Parallelism in the ADI

## An ADI Example in Python
