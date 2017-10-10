---
layout: default
title: "The Alternating Direction Implicit Method"
date: 2013-12-03
tags: wip popular
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

$$\frac{U_{j,i}^{n+1/2} - U_{j,i}^n}{\Delta t / 2} =
\frac{D}{2 \Delta x^2} \left( U_{j+1,i}^{n+1/2} - 2 U_{j,i}^{n+1/2} +
U_{j-1,i}^{n+1/2} \right) +
\frac{D}{2 \Delta y^2} \left( U_{j,i+1}^n - 2 U_{j,i}^n + U_{j,i-1}^n \right) +
\Delta t f(U_{j,i}^n),$$

$$\frac{U_{j,i}^{n+1} - U_{j,i}^{n+1/2}}{\Delta t / 2} =
\frac{D}{2 \Delta x^2} \left( U_{j+1,i}^{n+1/2} - 2 U_{j,i}^{n+1/2} +
U_{j-1,i}^{n+1/2} \right) +
\frac{D}{2 \Delta y^2} \left( U_{j,i+1}^{n+1} - 2 U_{j,i}^{n+1} +
U_{j,i-1}^{n+1} \right) +
\Delta t f(U_{j,i}^{n+1/2}).$$

## The ADI Linear Systems

We define $\alpha_x = \frac{D \Delta t}{\Delta x^2}$ and $\alpha_y = \frac{D
\Delta t}{\Delta y^2}$.
Note that $2 \sigma_x = \alpha_x$ and $2 \sigma_y = \alpha_y$.

Reordering both stencils, we obtain the following systems of linear equations

$$
-\alpha_x U_{j-1,i}^{n+1/2} + (1+2\alpha_x) U_{j,i}^{n+1/2} - \alpha_x
U_{j+1,i}^{n+1/2} =
\alpha_y U_{j,i-1}^n + (1-2\alpha_y) U_{j,i}^n + \alpha_y U_{j,i+1}^n + \Delta t
f(U_{j,i}^n)
$$

$$
-\alpha_y U_{j,i+1}^{n+1} + (1+2\alpha_y) U_{j,i}^{n+1} - \alpha_y
U_{j,i-1}^{n+1} =
\alpha_x U_{j-1,i}^{n+1/2} + (1-2\alpha_x) U_{j,i}^{n+1/2} + \alpha_x
U_{j+1,i}^{n+1/2} + \Delta t f(U_{j,i}^{n+1/2})
$$

### Family of Linear Systems in the $x$-Direction

Consider the stencil pictured in
[Dehghan Figure
1](http://www.sciencedirect.com/science/article/pii/S0377042700004520#FIG1)
and let us think of the matrix defined by $\left(U_{j,i}^n\right)$
($j=0,\ldots,J-1$, $i=0\ldots,I-1$, $n$ held constant) as a concentration plane.

We realize that for fixed index $i$ the first of our two equations above define
a
system of linear equations similar to the linear system we obtained for
[CN with one space dimension](http://georg.io/2013/12/03/Crank_Nicolson.html#reo
rdering_stencil_into_linear_system).
As for CN with one space dimension we need to amend certain entries of our
linear system to
accommodate our Neumann boundary conditions on the edges of our grid.
In fact we need to accommodate boundary conditions in both the $x$- and
$y$-direction.

Our first equation does not need to be modified for grid points that lie
"within" the
spatial part of our grid, i.e. $(j,i,n)$ for $j=1,\ldots,J-2$, $i=1,\ldots,I-2$,
and $n=0,1,2,\ldots$:

$$
-\alpha_x U_{j-1,i}^{n+1/2} + (1+2\alpha_x) U_{j,i}^{n+1/2} - \alpha_x
U_{j+1,i}^{n+1/2} =
\alpha_y U_{j,i-1}^n + (1-2\alpha_y) U_{j,i}^n + \alpha_y U_{j,i+1}^n + \Delta t
f(U_{j,i}^n).
$$

Let us take a look at what happens for grid points on the far left and right
with $j=0,J-1$ and $i=1,\ldots,I-2$:

$$
j=0:~ (1+\alpha_x) U_{0,i}^{n+1/2} - \alpha_x U_{1,i}^{n+1/2} =
\alpha_y U_{0,i-1}^n + (1-2\alpha_y) U_{0,i}^n + \alpha_y U_{0,i+1}^n + \Delta t
f(U_{0,i}^n),
$$

$$
j=J-1:~ -\alpha_x U_{J-2,i}^{n+1/2} + (1+\alpha_x) U_{J-1,i}^{n+1/2} =
\alpha_y U_{J-1,i-1}^n + (1-2\alpha_y) U_{J-1,i}^n + \alpha_y U_{J-1,i+1}^n +
\Delta t f(U_{J-1,i}^n).
$$

On the left-hand side of the linear system defined by these three expressions
(imagine varying $j=0,1,\ldots,J-1$)
we can already make out essentially the same matrix as we constructed for
[CN with one space dimension](http://georg.io/2013/12/03/Crank_Nicolson.html#reo
rdering_stencil_into_linear_system).
Note that this matrix is the same for all indeces $i=0,1,\ldots,I-1$.

This completes incorporating our boundary conditions in the $x$-direction.
Before we move on to doing the same with our boundary conditions in the
$y$-direction let us
state clearly that our $(x,y)$-concentration plane has
[right-handed orientation](http://en.wikipedia.org/wiki/Cartesian_coordinate_sys
tem#In_two_dimensions)
meaning that $x$ increases *left to right* and $y$ increases *bottom to top*.
Our grid has the same orientation so that index $j$ incrases *left to right* and
index $i$ increases *bottom to top*.

Let us now take a look at this equation for $j=1,\ldots,J-2$ and $i=0$ (bottom)
and $i=I-1$ (top):

$$
i=0:~ -\alpha_x U_{j-1,0}^{n+1/2} + (1+2\alpha_x) U_{j,0}^{n+1/2} - \alpha_x
U_{j+1,0}^{n+1/2} =
(1-\alpha_y) U_{j,0}^n + \alpha_y U_{j,1}^n + \Delta t f(U_{j,0}^n),
$$

$$
i=I-1:~ -\alpha_x U_{j-1,I-1}^{n+1/2} + (1+2\alpha_x) U_{j,I-1}^{n+1/2} -
\alpha_x U_{j+1,I-1}^{n+1/2} =
\alpha_y U_{j,I-2}^n + (1-\alpha_y) U_{j,I-1}^n + \Delta t f(U_{j,I-1}^n).
$$

To rewrite these equations in compact matrix notation, let us define a
horizontal slice (*left to right*)
through our concentration plane as vector

$$\mathbf{U}_{x,i}^n = \begin{bmatrix}U_{0,i}^n, & \ldots, & U_{J-1,i}^n
\end{bmatrix}$$

We can now combine all of the above and write the first of our two families of
ADI linear systems compactly:

$$A \mathbf{U}_{x,i}^{n+1/2} = \mathbf{b}_i + \mathbf{f}\left( \Delta t
\mathbf{U}_{x,i}^n \right),~i=0,\ldots,I-1,$$

where

$$A = \begin{bmatrix}
1+\alpha_x & -\alpha_x & 0 & 0 & 0 & \cdots & 0 & 0 & 0 & 0\\\\
-\alpha_x & 1+2\alpha_x & -\alpha_x & 0 & 0 & \cdots & 0 & 0 & 0 & 0 \\\\
0 & -\alpha_x & 1+2\alpha_x & -\alpha_x & \cdots & 0 & 0 & 0 & 0 & 0 \\\\
0 & 0 & \ddots & \ddots & \ddots & \ddots & 0 & 0 & 0 & 0 \\\\
0 & 0 & 0 & 0 & 0 & 0 & 0 & -\alpha_x & 1+2\alpha_x & -\alpha_x \\\\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -\alpha_x & 1+\alpha_x
\end{bmatrix}.$$

The form of vector $\mathbf{b}_i$ depends on $i$:

$$i=I-1:~ \mathbf{b}_{I-1} =
\begin{bmatrix}
(1-\alpha_y) U_{0,I-1}^n + \alpha_y U_{0,I-2}^n \\\\
\vdots \\\\
(1-\alpha_y) U_{J-1,I-1}^n + \alpha_y U_{J-1,I-2}^n
\end{bmatrix}$$

$$i=I-2,\ldots,1:~ \mathbf{b}_i =
\begin{bmatrix}
\alpha_y U_{0,i+1}^n + (1-2\alpha_y) U_{0,i}^n + \alpha_y U_{0,i-1}^n \\\\
\vdots \\\\
\alpha_y U_{J-1,i+1}^n + (1-2\alpha_y) U_{J-1,i}^n + \alpha_y U_{J-1,i-1}^n
\end{bmatrix}$$

$$i=0:~ \mathbf{b}_0 =
\begin{bmatrix}
\alpha_y U_{0,1}^n + (1-\alpha_y) U_{0,0}^n \\\\
\vdots \\\\
\alpha_y U_{J-1,1}^n + (1-\alpha_y) U_{J-1,0}^n
\end{bmatrix}.$$

The reaction term vector is

$$\mathbf{f}\left( \Delta t \mathbf{U}_{x,i}^n \right) =
\begin{bmatrix}
\Delta t f(U_{0,i}^n), & \ldots, & \Delta t f(U_{J-1,i}^n)
\end{bmatrix}$$

### Family of Linear Systems in the $y$-Direction

Let us first define a vertical (*from top to bottom*) slice through our
concentration plane
(note our comment on
[right-handedness](http://en.wikipedia.org/wiki/Cartesian_coordinate_system#In_t
wo_dimensions) above) as:

$$\mathbf{U}_{y,j}^n = \begin{bmatrix}U_{j,I-1}^n, & U_{j,I-2}^n, & \ldots, &
U_{j,0}^n \end{bmatrix}$$

Following an equivalent procedure to above for the second family of ADI linear
systems we obtain:

$$C \mathbf{U}_{y,j}^{n+1} = \mathbf{d}_j + \mathbf{f}\left( \Delta t
\mathbf{U}_{y,j}^{n+1/2} \right),~j=0,\ldots,J-1,$$

where

$$C = \begin{bmatrix}
1+\alpha_y & -\alpha_y & 0 & 0 & 0 & \cdots & 0 & 0 & 0 & 0\\\\
-\alpha_y & 1+2\alpha_y & -\alpha_y & 0 & 0 & \cdots & 0 & 0 & 0 & 0 \\\\
0 & -\alpha_y & 1+2\alpha_y & -\alpha_y & \cdots & 0 & 0 & 0 & 0 & 0 \\\\
0 & 0 & \ddots & \ddots & \ddots & \ddots & 0 & 0 & 0 & 0 \\\\
0 & 0 & 0 & 0 & 0 & 0 & 0 & -\alpha_y & 1+2\alpha_y & -\alpha_y \\\\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -\alpha_y & 1+\alpha_y
\end{bmatrix}.$$

The form of vector $\mathbf{d}_j$ depends on $j$:

$$j=0:~ \mathbf{d}_0 =
\begin{bmatrix}
(1-\alpha_x) U_{0,I-1}^{n+1/2} + \alpha_x U_{1,I-1}^{n+1/2} \\\\
(1-\alpha_x) U_{0,I-2}^{n+1/2} + \alpha_x U_{1,I-2}^{n+1/2} \\\\
\vdots \\\\
(1-\alpha_x) U_{0,0}^{n+1/2} + \alpha_x U_{1,0}^{n+1/2}
\end{bmatrix}$$

$$j=1,\ldots,J-2:~ \mathbf{d}_j =
\begin{bmatrix}
\alpha_x U_{j-1,I-1}^{n+1/2} + (1-2\alpha_x) U_{j,I-1}^{n+1/2} + \alpha_x
U_{j+1,I-1}^{n+1/2} \\\\
\alpha_x U_{j-1,I-2}^{n+1/2} + (1-2\alpha_x) U_{j,I-2}^{n+1/2} + \alpha_x
U_{j+1,I-2}^{n+1/2} \\\\
\vdots \\\\
\alpha_x U_{j-1,0}^{n+1/2} + (1-2\alpha_x) U_{j,0}^{n+1/2} + \alpha_x
U_{j+1,0}^{n+1/2}
\end{bmatrix}$$

$$j=J-1:~ \mathbf{d}_{J-1} =
\begin{bmatrix}
\alpha_x U_{J-2,I-1}^{n+1/2} + (1-\alpha_x) U_{J-1,I-1}^{n+1/2} \\\\
\alpha_x U_{J-2,I-2}^{n+1/2} + (1-\alpha_x) U_{J-1,I-2}^{n+1/2} \\\\
\vdots \\\\
\alpha_x U_{J-2,0}^{n+1/2} + (1-\alpha_x) U_{J-1,0}^{n+1/2}
\end{bmatrix}$$

The reaction term vector is

$$\mathbf{f}\left( \Delta t \mathbf{U}_{y,j}^{n+1/2} \right) =
\begin{bmatrix}
\Delta t f(U_{j,I-1}^{n+1/2}), & \Delta t f(U_{j,I-2}^{n+1/2}), & \ldots, &
\Delta t f(U_{j,0}^{n+1/2})
\end{bmatrix}$$

## Parallelism in the ADI

To summarize, the ADI stencil generates two families of linear systems that we
need to solve iteratively:

$$A \mathbf{U}_{x,i}^{n+1/2} = \mathbf{b}_i + \mathbf{f}\left( \Delta t
\mathbf{U}_{x,i}^n \right),~i=0,\ldots,I-1,$$

$$C \mathbf{U}_{y,j}^{n+1} = \mathbf{d}_j + \mathbf{f}\left( \Delta t
\mathbf{U}_{y,j}^{n+1/2} \right),~j=0,\ldots,J-1.$$

Upon closer inspection, we realize that we can solve the $I$ linear systems of
the first family in parallel.
To see this let us take a look at two arbitrary linear systems out of this
family:

$$A \mathbf{U}_{x,i_1}^{n+1/2} = \mathbf{b}_{i_1} + \mathbf{f}\left( \Delta t
\mathbf{U}_{x,i_1}^n \right),$$

$$A \mathbf{U}_{x,i_2}^{n+1/2} = \mathbf{b}_{i_2} + \mathbf{f}\left( \Delta t
\mathbf{U}_{x,i_2}^n \right).$$

As we can see, there is no interdependence between the linear systems for
indeces $i_1$ and $i_2$ that would prevent
us from solving these in parallel.

The same reasoning can be applied to our second family of linear systems.
We can solve the linear systems of all indeces $j$ of that family in parallel.

## An ADI Example in Python


    import numpy
    from matplotlib import pyplot


    J = 5
    I = 10
    
    print numpy.array([[str((j,i)) for j in range(J)] for i in range(I-1,-1,-1)])

    [['(0, 9)' '(1, 9)' '(2, 9)' '(3, 9)' '(4, 9)']
     ['(0, 8)' '(1, 8)' '(2, 8)' '(3, 8)' '(4, 8)']
     ['(0, 7)' '(1, 7)' '(2, 7)' '(3, 7)' '(4, 7)']
     ['(0, 6)' '(1, 6)' '(2, 6)' '(3, 6)' '(4, 6)']
     ['(0, 5)' '(1, 5)' '(2, 5)' '(3, 5)' '(4, 5)']
     ['(0, 4)' '(1, 4)' '(2, 4)' '(3, 4)' '(4, 4)']
     ['(0, 3)' '(1, 3)' '(2, 3)' '(3, 3)' '(4, 3)']
     ['(0, 2)' '(1, 2)' '(2, 2)' '(3, 2)' '(4, 2)']
     ['(0, 1)' '(1, 1)' '(2, 1)' '(3, 1)' '(4, 1)']
     ['(0, 0)' '(1, 0)' '(2, 0)' '(3, 0)' '(4, 0)']]



    L_x = 1.
    L_y = 2.
    T = 100.
    
    J = 100
    I = 200
    N = 1000
    
    dx = float(L_x)/float(J-1)
    dy = float(L_y)/float(I-1)
    dt = float(T)/float(N-1)
    
    x_grid = numpy.array([j*dx for j in range(J)])
    y_grid = numpy.array([i*dy for i in range(I)])
    t_grid = numpy.array([n*dt for n in range(N)])


    D_U = 0.1
    D_V = 10.


    tot_protein = 2.26


    no_high = 10
    U =  numpy.array([[0.1 if (I-1)-i+1 > no_high else 2.0 for j in range(J)] for i in range(I)])
    
    U_protein = sum(sum(U))*dx*dy
    V_protein_px = float(tot_protein-U_protein)/float(I*J*dx*dy)
    V = numpy.array([[V_protein_px for i in range(0,J)] for i in range(I)])
    
    print 'Initial protein mass', (sum(sum(U))+sum(sum(V)))*(dx*dy)

    Initial protein mass 2.26



    fig, ax = pyplot.subplots()
    ax.set_xlim(left=0., right=(J-1)*dx)
    ax.set_ylim(bottom=0., top=(I-1)*dy)
    heatmap = ax.pcolor(x_grid, y_grid, U, vmin=0., vmax=2.1)
    colorbar = pyplot.colorbar(heatmap)
    colorbar.set_label('concentration U')


![png](https://dl.dropboxusercontent.com/u/129945779/georgio/2013-12-03-Alternating_Direction_Implicit_Method_files/2013-12-03-Alternating_Direction_Implicit_Method_25_0.png)



    alpha_x_U = D_U*dt/(dx*dx)
    alpha_y_U = D_U*dt/(dy*dy)
    
    alpha_x_V = D_V*dt/(dx*dx)
    alpha_y_V = D_V*dt/(dy*dy)


    A_U = numpy.diagflat([-alpha_x_U for j in range(J-1)], -1)+\
        numpy.diagflat([1.+alpha_x_U]+[1.+2.*alpha_x_U for j in range(J-2)]+[1.+alpha_x_U], 0)+\
        numpy.diagflat([-alpha_x_U for j in range(J-1)], 1)
        
    C_U = numpy.diagflat([-alpha_y_U for i in range(I-1)], -1)+\
          numpy.diagflat([1.+alpha_y_U]+[1.+2.*alpha_y_U for i in range(I-2)]+[1.+alpha_y_U], 0)+\
          numpy.diagflat([-alpha_y_U for i in range(I-1)], 1)
            
    A_V = numpy.diagflat([-alpha_x_V for j in range(J-1)], -1)+\
        numpy.diagflat([1.+alpha_x_V]+[1.+2.*alpha_x_V for j in range(J-2)]+[1.+alpha_x_V], 0)+\
        numpy.diagflat([-alpha_x_V for j in range(J-1)], 1)
        
    C_V = numpy.diagflat([-alpha_y_V for i in range(I-1)], -1)+\
          numpy.diagflat([1.+alpha_y_V]+[1.+2.*alpha_y_V for i in range(I-2)]+[1.+alpha_y_V], 0)+\
          numpy.diagflat([-alpha_y_V for i in range(I-1)], 1)


    f_vec = lambda U, V: numpy.multiply(dt, numpy.subtract(numpy.multiply(V, 
                         numpy.add(k0, numpy.divide(numpy.multiply(U,U), numpy.add(1., numpy.multiply(U,U))))), U))
    
    k0 = 0.067


    b_t_stencil_U = numpy.array([[(1.-alpha_y_U) for j in range(J)],
                                 [alpha_y_U for j in range(J)]])
    b_c_stencil_U = numpy.array([[alpha_y_U for j in range(J)],
                                 [1.-2.*alpha_y_U for j in range(J)],
                                 [alpha_y_U for j in range(J)]])
    b_b_stencil_U = numpy.array([[alpha_y_U for j in range(J)],
                                 [(1.-alpha_y_U) for j in range(J)]])
    
    b_t_stencil_V = numpy.array([[(1.-alpha_y_V) for j in range(J)],
                                 [alpha_y_V for j in range(J)]])
    b_c_stencil_V = numpy.array([[alpha_y_V for j in range(J)],
                                 [1.-2.*alpha_y_V for j in range(J)],
                                 [alpha_y_V for j in range(J)]])
    b_b_stencil_V = numpy.array([[alpha_y_V for j in range(J)],
                                 [(1.-alpha_y_V) for j in range(J)]])
    
    f_curr = f(U,V)
    
    def b_U(i):
        if i <= I-2 and i >= 1:
            U_y = U[[i+1, i, i-1], :]
            return numpy.add(U_y+b_c_stencil_U, f_curr[[i+1, i, i-1], :]).sum(axis=0)
        elif i == I-1:
            U_y = U[[I-1, I-2], :]
            return numpy.add(U_y+b_t_stencil_U, f_curr[[I-1, I-2], :]).sum(axis=0)
        elif i == 0:
            U_y = U[[1, 0], :]
            return numpy.add(U_y+b_b_stencil_U, f_curr[[1, 0], :]).sum(axis=0)
        
    def b_V(i):
        if i <= I-2 and i >= 1:
            V_y = V[[i+1, i, i-1], :]
            return numpy.add(V_y+b_c_stencil_V, f_curr[[i+1, i, i-1], :]).sum(axis=0)
        elif i == I-1:
            V_y = V[[I-1, I-2], :]
            return numpy.add(V_y+b_t_stencil_V, f_curr[[I-1, I-2], :]).sum(axis=0)
        elif i == 0:
            V_y = V[[1, 0], :]
            return numpy.add(V_y+b_b_stencil_V, f_curr[[1, 0], :]).sum(axis=0)


    numpy.linalg.solve(A, b_U(0))

    array([  54.714724  ,   55.25042972,   56.33706168,   57.98426118,
             60.20635023,   63.02986304,   66.47710598,   70.58939494,
             75.40022933,   80.9673746 ,   87.34757575,   94.6048406 ,
            102.81416575,  112.05900553,  122.43350823,  134.02121346,
            146.96230295,  160.84817102,  159.56797494,  159.90177877,
            161.85177784,  165.28713763,  170.37389138,  177.18500712,
            185.78990942,  196.23310272,  208.6632231 ,  215.11512097,
            223.72896304,  234.61098759,  247.20901905,  262.31458036,
            280.08164015,  300.58373665,  298.77153646,  299.98741957,
            304.23993722,  311.55700462,  320.8107233 ,  333.32209491,
            349.21872655,  368.63959346,  391.80570462,  355.2372413 ,
            322.27276918,  292.57921192,  265.78799108,  241.66002915,
            218.84403963,  198.19823191,  179.56039514,  162.74055687,
            147.56727522,  133.70166181,  121.18129297,  109.88385729,
             99.67876762,   90.47745603,   82.17679059,   74.70150841,
             67.97541513,   61.9299528 ,   56.49879025,   51.61670521,
             47.244502  ,   43.34161898,   39.82944649,   36.71101767,
             33.75639149,   31.13360733,   28.31416878,   25.77110054,
             23.47848144,   21.39678548,   19.52095204,   17.83186103,
             16.31229588,   14.9467679 ,   13.7213585 ,   12.62357553,
             11.64223111,   10.76726919,    9.98982488,    9.3019734 ,
              8.69669468,    8.16600012,    7.70630888,    7.31292036,
              6.98183752,    6.63212981,    6.33718047,    6.08211195,
              5.76917204,    5.50280458,    5.28029158,    5.09935233,
              4.95815374,    4.85355401,    4.78266555,    4.7465165 ])


