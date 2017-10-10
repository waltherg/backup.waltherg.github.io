---
layout: default
date: 2013-12-04
title: "The Crank-Nicolson Method for Convection-Diffusion Systems"
tags: Python Numpy Numerical-Analysis Partial-Differential-Equation popular
---

# The Crank-Nicolson Method for Convection-Diffusion Systems

Here we extend [our discussion and
implementation](http://georg.io/2013/12/03/Crank_Nicolson.html) of the Crank-
Nicolson (CN) method
to [convection-diffusion
systems](http://en.wikipedia.org/wiki/Convection%E2%80%93diffusion_equation).

To clarify nomenclature, there is a physically important difference between
[convection and advection](http://physics.stackexchange.com/a/24494/20924).
Since we are interested in the transport of a protein suspended in a
(semi)liquid medium, we will use
the term *advection* (as opposed to *convection*) in the following discussion.

## Our Advection-Diffusion Equation

We study the following advection-diffusion equation:

$$\frac{\partial u}{\partial t} = D \frac{\partial^2 u}{\partial x^2} +
\frac{\partial}{\partial x} \left(a u \right) + f(u),$$

$$\frac{\partial u}{\partial x}\Bigg|_{x = 0, L} = 0,$$

with concentration $u$, diffusion coefficient $D$, advection velocity $a$
(velocity of the medium), reaction term $f$,
and domain length $L$.
Our [Neumann boundary
conditions](http://en.wikipedia.org/wiki/Neumann_boundary_condition) make
certain that no amount of $u$
enters or leaves our domain.

## Crank-Nicolson Stencil for the Advection Term

We use the same grid as 
[before](http://georg.io/2013/12/03/Crank_Nicolson.html#finite_difference_methods).
Also see [our earlier discussion](http://georg.io/2013/12/03/Crank_Nicolson.html#the_cranknicolson_stencil) of finite difference stencils.

As described, for instance, in [Trefethen Table
3.2.1](http://people.maths.ox.ac.uk/trefethen/3all.pdf) the Crank-Nicolson
stencil
for the advection term takes the following form (assuming that $a$ is constant)

$$\frac{\partial (au)}{\partial x} \approx \frac{a}{4 \Delta x} \left( U_{j+1}^n
- U_{j-1}^n + U_{j+1}^{n+1} - U_{j-1}^{n+1} \right).$$

Extending our [previous CN stencil](http://georg.io/2013/12/03/Crank_Nicolson.ht
ml#the_cranknicolson_stencil) with this expression
and applying the resulting, extended stencil to our advection-diffusion equation
in grid point $(j,n)$ we obtain:

$$\frac{U_j^{n+1} - U_j^n}{\Delta t} = \frac{D}{2 \Delta x^2} \left( U_{j+1}^n -
2 U_j^n + U_{j-1}^n + U_{j+1}^{n+1} - 2 U_j^{n+1} + U_{j-1}^{n+1}\right) +
\frac{a}{4 \Delta x} \left( U_{j+1}^n - U_{j-1}^n + U_{j+1}^{n+1} -
U_{j-1}^{n+1} \right) +
f(U_j^n).$$

## Extended Linear System with Advection Term

[As before](http://georg.io/2013/12/03/Crank_Nicolson.html#reordering_stencil_in
to_linear_system),
we define $\sigma = \frac{D \Delta t}{2 \Delta x^2}$ and $\rho = \frac{a \Delta
t}{4 \Delta x}$ and rearrange the above approximation
of our advection-diffusion equation:

$$(-\sigma+\rho) U_{j-1}^{n+1} + (1+2\sigma) U_j^{n+1} -(\sigma+\rho)
U_{j+1}^{n+1} =
(\sigma-\rho) U_{j-1}^n + (1-2\sigma) U_j^n + (\sigma+\rho) U_{j+1}^n + \Delta t
f(U_j^n),~j=1,\ldots,J-2$$

This equation holds for all grid points with spatial indices $j=1,\ldots,J-2$.
On the two boundaries of our spatial grid we apply the above Neumann boundary
conditions that dictate

$$U_{-1}^n = U_0^n,~U_{J-1}^n = U_J^n,~n=0,1,\ldots,$$

resulting in the following amended expressions for $j=0$ and $j=J-1$:

$$j=0:~ (1+\sigma+\rho) U_0^{n+1} -(\sigma+\rho) U_{1}^{n+1} =
(1-\sigma-\rho) U_0^n + (\sigma+\rho) U_{1}^n + \Delta t f(U_0^n),$$

$$j=J-1:~ (-\sigma+\rho) U_{J-2}^{n+1} + (1+\sigma-\rho) U_{J-1}^{n+1} =
(\sigma-\rho) U_{J-2}^n + (1-\sigma+\rho) U_{J-1}^n + \Delta t f(U_{J-1}^n).$$

We use the same vector notation, $\mathbf{U}^n$ and $\mathbf{f}^n$, and matrix
names $A$ and $B$
[as before](http://georg.io/2013/12/03/Crank_Nicolson.html#reordering_stencil_in
to_linear_system)
and can now write the above equations as a linear system in matrix notation:

$$A \mathbf{U}^{n+1} = B \mathbf{U}^n + \mathbf{f}^n,$$

where the tridiagonal matrix $A$ has the following vector of length $J$ on its
diagonal

$$\begin{bmatrix} 1+\sigma+\rho, & 1+2\sigma, & \ldots, & 1+2\sigma, &
1+\sigma-\rho \end{bmatrix},$$

the following vector of length $J-1$ on its first
[superdiagonal](http://en.wikipedia.org/wiki/Diagonal#Matrices)

$$\begin{bmatrix} -(\sigma+\rho), & \ldots, & -(\sigma+\rho) \end{bmatrix},$$

and the following vector of length $J-1$ on its first
[subdiagonal](http://en.wikipedia.org/wiki/Diagonal#Matrices):

$$\begin{bmatrix} (-\sigma+\rho), & \ldots, & (-\sigma+\rho) \end{bmatrix}.$$

The tridiagonal matrix $B$ has the following vector of length $J$ on its
diagonal

$$\begin{bmatrix} 1-\sigma-\rho, & 1-2\sigma, & \ldots, & 1+2\sigma, &
1-\sigma+\rho \end{bmatrix},$$

the following vector of length $J-1$ on its first
[superdiagonal](http://en.wikipedia.org/wiki/Diagonal#Matrices)

$$\begin{bmatrix} (\sigma+\rho), & \ldots, & (\sigma+\rho) \end{bmatrix},$$

the following vector of length $J-1$ on its first
[subdiagonal](http://en.wikipedia.org/wiki/Diagonal#Matrices)

$$\begin{bmatrix} (\sigma-\rho), & \ldots, & (\sigma-\rho) \end{bmatrix},$$

## An Example in Python

Let us implement the above numerical scheme for the same example
[we discussed previously](http://georg.io/2013/12/03/Crank_Nicolson.html#a_crank
nicolson_example_in_python).

Construction of the linear system we solve to integrate our advection-diffusion
system numerically over one time step
is very similar to our [previous linear system](http://georg.io/2013/12/03/Crank
_Nicolson.html#reordering_stencil_into_linear_system).

We therefore expect, that we can reuse our [previous code](http://georg.io/2013/
12/03/Crank_Nicolson.html#a_cranknicolson_example_in_python)
and amend merely matrices `A_u`, `B_u`, `A_v`, and `B_v` as described above.

### Our Python Code

Here is just a copy-and-paste of
[our previous code](http://georg.io/2013/12/03/Crank_Nicolson.html#a_cranknicols
on_example_in_python).
Refer to that post for explanations.

All that changes in this code segment compared with [before](http://georg.io/201
3/12/03/Crank_Nicolson.html#a_cranknicolson_example_in_python)
are our definition of `rho` and construction of matrices `A_u`, `B_u`, `A_v`,
`B_v`.
(We actually also change `J` and `N` - see comment below)

Note that `rho` is the same for both protein species `U` and `V`.

As advection velocity `a` we choose a negative value so that the fluid carrying
proteins `U` and `V` flows from right to left.

    import numpy
    from matplotlib import pyplot
    numpy.set_printoptions(precision=3)
    
    L = 1.
    J = 500
    dx = float(L)/float(J-1)
    x_grid = numpy.array([j*dx for j in range(J)])
    
    T = 200
    N = 2000
    dt = float(T)/float(N-1)
    t_grid = numpy.array([n*dt for n in range(N)])
    
    D_v = float(10.)/float(100.)
    D_u = 0.01 * D_v
    
    k0 = 0.067
    f_vec = lambda U, V: numpy.multiply(dt, numpy.subtract(numpy.multiply(V, 
                         numpy.add(k0, numpy.divide(numpy.multiply(U,U), numpy.add(1., numpy.multiply(U,U))))), U))
     
    sigma_u = float(D_u*dt)/float(2.*dx*dx)
    sigma_v = float(D_v*dt)/float(2.*dx*dx)
    
    a = -0.0003
    rho = float(a*dt)/float(4.*dx)
    
    total_protein = 2.26
    
    no_high = 10
    U = numpy.array([0.1 for i in range(no_high,J)] + [2. for i in range(0,no_high)])
    V = numpy.array([float(total_protein-dx*sum(U))/float(J*dx) for i in range(0,J)])
    
    A_u = numpy.diagflat([-sigma_u+rho for i in range(J-1)], -1) +\
          numpy.diagflat([1.+sigma_u+rho]+[1.+2.*sigma_u for i in range(J-2)]+[1.+sigma_u-rho]) +\
          numpy.diagflat([-(sigma_u+rho) for i in range(J-1)], 1)
            
    B_u = numpy.diagflat([sigma_u-rho for i in range(J-1)], -1) +\
          numpy.diagflat([1.-sigma_u-rho]+[1.-2.*sigma_u for i in range(J-2)]+[1.-sigma_u+rho]) +\
          numpy.diagflat([sigma_u+rho for i in range(J-1)], 1)
            
    A_v = numpy.diagflat([-sigma_v+rho for i in range(J-1)], -1) +\
          numpy.diagflat([1.+sigma_v+rho]+[1.+2.*sigma_v for i in range(J-2)]+[1.+sigma_v-rho]) +\
          numpy.diagflat([-(sigma_v+rho) for i in range(J-1)], 1)
            
    B_v = numpy.diagflat([sigma_v-rho for i in range(J-1)], -1) +\
          numpy.diagflat([1.-sigma_v-rho]+[1.-2.*sigma_v for i in range(J-2)]+[1.-sigma_v+rho]) +\
          numpy.diagflat([sigma_v+rho for i in range(J-1)], 1)
            
    U_record = []
    V_record = []
    
    U_record.append(U)
    V_record.append(V)
    
    for ti in range(1,N):
        U_new = numpy.linalg.solve(A_u, B_u.dot(U) + f_vec(U,V))
        V_new = numpy.linalg.solve(A_v, B_v.dot(V) - f_vec(U,V))
        
        U = U_new
        V = V_new
        
        U_record.append(U)
        V_record.append(V)


We notice that the numerical stability of this scheme is greatly dependent on
the magnitude of `a`.

As was discussed by [Walther *et
al.*](http://link.springer.com/article/10.1007%2Fs11538-012-9766-5),
preservation of total protein mass in a certain regime near `total_protein =
2.26` is important to
preserve the pattern-forming pattern of the
[original system (excluding the advection term)](http://georg.io/2013/12/03/Cran
k_Nicolson.html#a_cranknicolson_example_in_python).

Notice that even though we increased `J = 500` and `N = 2000` for a much finer
numerical grid
compared with
[before](http://georg.io/2013/12/03/Crank_Nicolson.html#specify_grid)
(`J = 100`, `N = 1000`), we still lose some protein mass to numerical errors.

Initial protein mass:

    sum(numpy.multiply(dx,U_record[0]) + numpy.multiply(dx,V_record[0]))

    2.259999999999998

Protein mass at the end of the simulation:

    sum(numpy.multiply(dx,U_record[-1]) + numpy.multiply(dx,V_record[-1]))

    2.2064888602225103

We can reduce the loss of protein mass by reducing the magnitude of `a` or
refining our grid even further
by increasing both `J` and `N`.

An important question to answer would be how much numerical accuracy we gain by
increasing `J` and `N`.

And here is a kymograph showing the time and space behaviour of `U`.

    U_record = numpy.array(U_record)
    V_record = numpy.array(V_record)
    
    fig, ax = pyplot.subplots()
    pyplot.xlabel('x'); pyplot.ylabel('t')
    heatmap = ax.pcolor(x_grid, t_grid, U_record, vmin=0., vmax=1.2)
    colorbar = pyplot.colorbar(heatmap)
    colorbar.set_label('concentration U')

![png](https://dl.dropboxusercontent.com/u/129945779/georgio/2013-12-04-Crank_Nicolson_Convection_Diffusion_files/2013-12-04-Crank_Nicolson_Convection_Diffusion_21_0.png)

