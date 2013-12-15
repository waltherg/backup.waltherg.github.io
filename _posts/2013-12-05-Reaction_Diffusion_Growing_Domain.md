---
layout: default
date: 2013-12-05
title: "Reaction-Diffusion System on a Growing Domain"
tags: wip
---

# Reaction-Diffusion Systems on Growing Domains

Here I will discuss the same reaction-diffusion system in one space dimension
[as I considered previously](http://georg.io/2013/12/03/Crank_Nicolson.html#a_cr
anknicolson_example_in_python).

While in my previous blog post I discussed integrating this system numerically
on a one-dimensional domain of
fixed length, I will here consider the case of a growing domain.

Reaction-diffusion systems, specifically those that give rise to Turing-type
patterns, have been discussed
extensively by [Edmund J Crampin in his PhD thesis](http://solo.bodleian.ox.ac.u
k/primo_library/libweb/action/dlDisplay.do?vid=OXVU1&docId=oxfaleph015225500)
and related peer-reviewed publications.

In their work, Crampin *et al.* laid out a framework for integrating growth into
reaction-diffusion systems
that lends terminology from [finite strain
theory](http://en.wikipedia.org/wiki/Finite_strain_theory).
In this framework, the volume elements of our space domain are subjected to the
growth velocity of the
system and the protein entities dissolved in these volume elements are
transported alongside.

In this blog post I will describe how our familiar reaction-diffusion equations
are altered by
the introduction of a growth velocity. I will then describe how to transform
these altered equations
to a form that is more amenable to numerical simulations.
And at the bottom of this blog post I will provide Python code that integrates
these equations.
In all of this I will follow the framework provided by Crampin *et al.*

## Reaction-Diffusion System

As a brief recap, this is the system we discussed
[previously](http://georg.io/2013/12/03/Crank_Nicolson.html)

$$\frac{\partial u}{\partial t} = D_u \frac{\partial^2 u}{\partial x^2} +
f(u,v),$$

$$\frac{\partial v}{\partial t} = D_v \frac{\partial^2 v}{\partial x^2} -
f(u,v),$$

with Neumann boundary conditions

$$\frac{\partial u}{\partial x}\Bigg|_{x=0,L} = 0,$$

$$\frac{\partial v}{\partial x}\Bigg|_{x=0,L} = 0,$$

where concentration variables $u$ and $v$ represent the active and incative
state of a protein respectively.

## Introducing Growth

As with the simpler heat equation or reaction-diffusion equation on a domain of
fixed size,
we can derive the differential form of our reaction-diffusion equation on a
growing domain from
a mass conservation equation.
During the derivation we make use of Divergence Theorem and some truncated
Taylor series.

If I understand correctly, this derivation can be summarized as an application
of the
[Reynolds Transport
Theorem](http://en.wikipedia.org/wiki/Reynolds_transport_theorem).

Suppose now that our domain grows at velocity $a(x,t)$, then the aforementioned
derivation
results in the following modified reaction-diffusion system:

$$ \frac{\partial u}{\partial t} + \frac{\partial}{\partial x} (u a) =
D_u \frac{\partial^2 u}{\partial x^2} u +  f(u,v),$$

$$ \frac{\partial v}{\partial t} + \frac{\partial}{\partial x} (v a) =
D_v \frac{\partial^2 v}{\partial x^2} u -  f(u,v).$$

Important here is that we fix the growth velocity of the origin, $x=0$, to zero
so that
one end of our domain does not move while the far right of our domain $x=L$ will
move
to the right when our domain grows.
We impose this condition on the velocity of the origin to ensure that we do not
model
movement / translation of our domain as a whole but rather growth.

Incorporating this into our boundary conditions and writing $L(t)$ for the
moving boundary on the right we obtain

$$\frac{\partial u}{\partial x}\Bigg|_{x=0,L(t)} = 0,$$

$$\frac{\partial v}{\partial x}\Bigg|_{x=0,L(t)} = 0.$$

## Material Coordinates

Let us label the infinitessimal volume elements of our initial domain with $X
\in [0, L(0)]$, where $L$ is the length of the domain
(undoubtedly, this is a woolly definition but I mean a tiny subinteval of $[0,
L]$ of the shape $[X-\delta X, X+\delta X]$ in some
limit $ \delta X \downarrow 0$ but I am not certain I make more sense stating
this).

If our system grows, these volume elements $X$ will migrate and expand in size.
To track the movement of $X$ we need to assign to each $X$ a unique position in
space $x$.

We will refer to $X$ as the material coordinate and to $x$ as the space
coordinate.

The space coordinate $x$ of the material coordinate $X$ at time $t$ is described
by the trajectory of $X$

$$x = G(X,t),$$

where $G(\cdot)$ tells us the position $x$ of volume element $X$ at time $t$.

Generally, we do not know much about the trajectory $G$ as parameters, such as
how fast a volume element travels, depend
on our system of interest, its system parameters and so forth.

We do, however, know what our domain looks like initially (it's simply the
interval $[0, L(0)]$) and so we impose
the following initial condition on $G$:

$$G(X,0) = X,$$

which just means that the initial position $x$ of the element labeled with $X$
is equal to $x$.

As we already mentioned above, we want to model growth of our system and not its
translation (movement in space).
We therefore impose the following boundary condition on $G$:

$$G(0,t) = 0,$$

which reads that the position $x$ of the origin always equals zero.

With this notation and these conditions we now have a simple way of static the
velocity at which our system grows
(growth velocity):

$$\frac{\partial G}{\partial t} = a(X,t).$$

Note that we this specifies a local growth velocity, $a(X,t)$, that may vary
over material coordinate $X$ and time $t$.
Hence, parts of our system may grow slower or faster than others - effectively
stretching and deforming our system
in potentially interesting ways (I suppose for proper *deformation* we would
need to work with more than one
space dimension).

In one final step, let us apply the [fundamental theorem of
calculus](http://en.wikipedia.org/wiki/Fundamental_theorem_of_calculus)
to the above differential equation for $G$ to obtain:

$$G(X,t) = G(X,0) + \int_0^t{a(X,s) ds}.$$

Note that in choosing an equation (a model) for $a(X,t)$ we will always have to
ensure that the resultant $G(X,t)$
(whether or not we can write a nice analytic expression for $G$) meets the above
initial and boundary conditions.

## Our Reaction-Diffusion System in Material Coordinates

Within the framework described by Crampin *et al.*, it would be inconvenient or
hard to approximate
the time evolution of our modified partial differential equation in space
coordinates $x$.

Therefore, we here transform our above diferential equation from space
coordinates $x$ to
material coordinates $X$.
The equations resulting from this transformation describe how our protein
concentrations behave
dynamically in the volume elements we label initially with $X$.

For the transformation, we substitute the identity $x = G(X,t)$ into the above
partial
differential equation and obtain

$$\frac{\partial u}{\partial t} = \frac{D_u}{G_X^2} \frac{\partial^2 u}{\partial
X^2} - \left( D_u \frac{G_{XX}}{G_X^3} + \frac{G_t}{G_X} \right) \frac{\partial
u}{\partial X} + f(u,v) - \frac{G_{Xt}}{G_X} u,$$

$$\frac{\partial v}{\partial t} = \frac{D_v}{G_X^2} \frac{\partial^2 v}{\partial
X^2} - \left( D_v \frac{G_{XX}}{G_X^3} + \frac{G_t}{G_X} \right) \frac{\partial
v}{\partial X} - f(u,v) - \frac{G_{Xt}}{G_X} v.$$

From our above equality $G_t(X,t) = a(X,t)$ we can derive $G_{tX} = a_X$ and,
assuming [$G_{tX} =
G_{Xt}$](http://en.wikipedia.org/wiki/Symmetry_of_second_derivatives),
we obtain $G_{Xt} = a_X$.
Let us further write $G_X(X,t) = g(X,t)$ and substituting this notation into our
equations we get

$$\frac{\partial u}{\partial t} = \frac{D_u}{g^2} \frac{\partial^2 u}{\partial
X^2} - \left( D_u \frac{g_X}{g^3} + \frac{a}{g} \right) \frac{\partial
u}{\partial X} + f(u,v) - \frac{a_X}{g} u,$$

$$\frac{\partial v}{\partial t} = \frac{D_v}{g^2} \frac{\partial^2 v}{\partial
X^2} - \left( D_v \frac{g_X}{g^3} + \frac{a}{g} \right) \frac{\partial
v}{\partial X} - f(u,v) - \frac{a_X}{g} v.$$

With this notation we need to enforce the following conditions on $g(X,t)$ and
$a(X,t)$ to ensure that the
resultant trajectory $G(X,t)$ meets the above initial and boundary conditions:

$$g(X,0) = 1,$$

$$a(0,t) = 0.$$

The former follows directly from $G(X,0) = X$ and the latter ensures that the
origin always has velocity zero
and therefore does not move.

We note that these two conditions do not necessarily fix the origin in $x=0$.
However this is irrelevant as we are interested in growth of our system and not
its absolute location in space.

## Specification of the Local Growth Velocity

After all these derivations and changes in notation we finally get to the fun
part of this problem:
specifying local growth velocities of our system.

This is where we can be creative and test the effects of different hypotheses.
What happens if our system simply grows at some constant velocity?
What happens if the growth velocity is a function of the protein concentrations
in our system?

In the scenario of constant growth the local growth velocity becomes

$$a(X,t) = a_0,~ a_0 > 0.$$

With this simple model we know the predetermined growth velocity for all
material positions $X$
and time $t$ and can therefore apply the above integral expression for $G(X,t)$:

$$G(X,t) = X + t a_0.$$

Let us check the units of this expression:
clearly $X$ and $t a_0$ need to have the same units.
Using the [square bracket notation for
units](http://physics.stackexchange.com/a/77691/20924), we can express
this as

$$[X] = [t a_0] = [t][X] / [t].$$

To meet this equivalence in units, let us rewrite our local growth velocity  as
$a(X,t) = X \alpha(X,t)$ where the units of $[\alpha(X,t)] = 1/[t]$ -
assigning to $\alpha(X,t)$ the meaning of a
[rate](http://en.wikipedia.org/wiki/Rate_(mathematics))
("per unit time").

In the remainder of this article we will refer to this rate $\alpha(X,t)$ as the
local growth velocity.

Using this redefinition of the local growth velocity, the above expression for
$G(X,t)$ for
constant growth becomes ($\alpha(X,t) = \alpha_0$)

$$G(X,t) = X + t X \alpha_0,~ \alpha_0 > 0.$$

Given this expression for $G(X,t)$ it is easy to work out all partial
derivatives that
we require in the above equations and
[solve the resulting convection-diffusion system
numerically](http://georg.io/2013/12/Crank_Nicolson_Convection_Diffusion/).

In general however we do not know the growth velocity for all material positions
$X$ and time $t$ beforehand
and therefore cannot work out an analytic epxression for the trajectory
$G(X,t)$.

One such scenario is presented if we assume that the growth velocity is an
increasing function of
the local protein concentration $u(X,t)$:

$$a(X,t) = X \alpha(u(X,t)),$$

and, for the sake of argument, let us assume that $\alpha(u(X,t)) = \alpha_0
u(X,t),~ \alpha_0 > 0$.

Apart from the initial condition, $u(X,t)$ is unknown beforehand and we are
therefore unable to
work out $G(X,t)$ analytically.

We do however know that

$$g(X,0) = 1,$$

and, assuming
[$G_{tX} =
G_{Xt}$](http://en.wikipedia.org/wiki/Symmetry_of_second_derivatives), we also
know that

$$g_t(X,t) = a_X(X,t),$$

giving us a spatially uncoupled partial differential equation for $g(X,t)$.

Thus, if we integrate numerically $g(X,t)$ alongside $u(X,t)$ and $v(X,t)$ we
should be able
to approximate $g(X,t)$ for all material positions $X$ and time $t$.

This defines a PDE system of three unknown variables where the protein part of
the system,
$u(X,t)$ and $v(X,t)$, and the growth part of the system, $g(X,t)$, are linked
through the
growth velocity $a(X,t)$ (or rather $\alpha(X,t)$ considering our above
comments):

$$\frac{\partial u}{\partial t} = \frac{D_u}{g^2} \frac{\partial^2 u}{\partial
X^2} - \left( D_u \frac{g_X}{g^3} + \frac{a}{g} \right) \frac{\partial
u}{\partial X} + f(u,v) - \frac{a_X}{g} u,$$

$$\frac{\partial v}{\partial t} = \frac{D_v}{g^2} \frac{\partial^2 v}{\partial
X^2} - \left( D_v \frac{g_X}{g^3} + \frac{a}{g} \right) \frac{\partial
v}{\partial X} - f(u,v) - \frac{a_X}{g} v,$$

$$\frac{\partial g}{\partial t} = \frac{\partial a}{\partial X},$$

taken with the initial and boundary conditions mentioned above.

## Numerical Integration: Grid and Stencils

To integrate this system we use the same time and space grid as
we constructed for [the Crank-Nicolson
method](http://georg.io/2013/12/Crank_Nicolson).
Since our system of proteins contains both an advection and a diffusion term, we
need to
make use of the stencil we developed for
[convection-diffusion
systems](http://georg.io/2013/12/Crank_Nicolson_Convection_Diffusion).

Let us now consider $g(X,t)$ on this time and space grid and write

$$g(j \Delta x, n \Delta t) = g_j^n,$$

giving us a system of uncoupled differential equations

$$\frac{\partial g_j^n}{\partial t} = a_X(j \Delta X, n \Delta t),~
j=0,\ldots,J-1.$$

Hence, in our numerical integration of this system we can make use of our Crank-
Nicolson-based method we
[derived earlier](http://georg.io/2013/12/Crank_Nicolson_Convection_Diffusion)
for $u$ and $v$ while we can rely on existing library methods that integrate
(ordinary) differential equations
for the $J$ equations stemming from $g$.

## Python Code

This
[IPython magic command](http://ipython.org/ipython-
doc/dev/interactive/notebook.html#plotting)
enables display of graphical output directly inside this notebook.


    %matplotlib inline

[As before](http://georg.io/2013/12/Crank_Nicolson_Convection_Diffusion),
we will use the tridiagonal matrix solver implemented in NumPy.
To integrate numeically the $J$ ordinary differential equations for $g_j$,
we make use of the
[`integrate`](http://docs.scipy.org/doc/scipy/reference/tutorial/integrate.html)
subpackage of
['scipy'](http://docs.scipy.org/doc/scipy/reference/index.html).


    import numpy
    from scipy import integrate
    from matplotlib import pyplot
    numpy.set_printoptions(precision=3)

The spatial part of our grid will be slightly different from the grid we
constructed for a [static domain](http://georg.io/2013/12/Crank_Nicolson).

Here, we will track all our variables (`U`, `V`, `a`, and `g`) in the midpoints
of `J` intervals.
Since we simulate our system in material coordinates, the length of each of
these
`J` intervals will equal `dX` throughout the integration.
We save these `J` midpoints in `X_grid`.

To track deformations of our space grid over time, we will also track movement
of
the `J+1` endpoints of each of our `J` grid midpoints.
We save these `J+1` endpoints in `x_grid`.

Note that the `J` midpoints that we are interested in do not move as we express
their positions in material coordinates.
However, the `J+1` endpoints will move as we express those in space coordinates.


    L = 1.
    J = 100
    dX = float(L)/float(J)
    X_grid = numpy.array([float(dX)/2. + j*dX for j in range(J)])
    x_grid = numpy.array([j*dX for j in range(J+1)])
    
    T = 200
    N = 1000
    dt = float(T)/float(N-1)
    t_grid = numpy.array([n*dt for n in range(N)])


    D_v = float(10.)/float(100.)
    D_u = 0.01 * D_v

[As before](http://georg.io/2013/12/Crank_Nicolson/), we express the
reaction term as a vector.


    k0 = 0.067
    f_vec = lambda U, V: numpy.multiply(dt, numpy.subtract(numpy.multiply(V, 
                         numpy.add(k0, numpy.divide(numpy.multiply(U,U), numpy.add(1., numpy.multiply(U,U))))), U))

Let us now write down the initial conditions for our variables.


    total_protein = 2.26
    
    no_high = 10
    U = numpy.array([0.1 for i in range(no_high,J)] + [2. for i in range(0,no_high)])
    V = numpy.array([float(total_protein-dX*sum(U))/float(J*dX) for i in range(0,J)])
    g = numpy.array([1. for j in range(J)])
    a = lambda U: numpy.array([0.001*X_grid[j]*U[j] for j in range(J)])

For concentrations `U` and `V` we choose the same initial conditions as we
did [previously]().

Note that we plot `U` and `V` against the `J` midpoints stored in `X_grid`.


    pyplot.ylim((0., 2.1))
    pyplot.xlabel('X'); pyplot.ylabel('concentration')
    pyplot.plot(X_grid, U)
    pyplot.plot(X_grid, V)
    pyplot.show()


![png]({{site.imgbase}}/2013-12-05-Reaction_Diffusion_Growing_Domain_files/2013-12-05-Reaction_Diffusion_Growing_Domain_27_0.png)


The initial condition on `g` is as described above and just a horizontal line.


    pyplot.plot(X_grid, g)
    pyplot.xlabel('X'); pyplot.ylabel('g')
    pyplot.show()


![png]({{site.imgbase}}/2013-12-05-Reaction_Diffusion_Growing_Domain_files/2013-12-05-Reaction_Diffusion_Growing_Domain_29_0.png)


Note that we need to write `a` as a function of `U` so that the values of
`a` are updated automatically when `U` changes.

Given the initial condition on `U`, `a(U)` looks as follows initially.


    pyplot.plot(X_grid, a(U))
    pyplot.xlabel('X'); pyplot.ylabel('a')
    pyplot.show()


![png]({{site.imgbase}}/2013-12-05-Reaction_Diffusion_Growing_Domain_files/2013-12-05-Reaction_Diffusion_Growing_Domain_31_0.png)


If you check again our equations in material coordinates you will notice that we
subtract a term involving $a_X$ from the reaction term in both differential
equations.

To simplify our notation further below we will combine this difference into
a modified reaction term.

Since we only specify $a(X,t)$ and not $a_X$ we need to approximate $a_X$ from
the current vector `a` with finite differences.

Let us introduce two helper vectors that we can use to approximate $a_X$
conveniently with
finite differences:


    # syntax to get one element in an array: http://stackoverflow.com/a/7332880/3078529
    a_left = lambda a: numpy.concatenate((a[1:J], a[J-1:J]))
    a_right = lambda a: numpy.concatenate((a[0:1], a[0:J-1]))

We write $a_j^n = a(\Delta X/2 + j \Delta X, n \Delta t)$ with $j=0,\ldots,J-1$,
and we therefore have no values for $a_{-1}^n$ nor $a_{J}^n$.

I have not thought deeply about the boundary conditions for $a(X,t)$ but for the
time
being I will simply say

$$a_{-1}^n = a_{0}^n,$$

$$a_{J-1}^n = a_J^n.$$

We will now approximate $a_X$ as follows:

$$\frac{\partial a}{\partial X} \approx \frac{a_{j+1}^n - a_{j-1}^n}{2 \Delta
X},$$

where of course we apply the above boundary conditions where appropriate.

Without thinking about this more deeply, I would be tempted to divide by $\Delta
X$ and not $2 \Delta X$
where the boundary conditions apply since e.g. for $j=0$ we get

$$\frac{\partial a}{\partial X} \approx \frac{a_1^n - a_0^n}{2 \Delta X},$$

so to me it would seem more natural to divide by $\Delta X$ rather than $2
\Delta X$.

This can be achieved with the following bit of code:

    f_vec_u = lambda U, V, g, a: numpy.subtract(f_vec(U, V),
    numpy.multiply(numpy.subtract(a_left(a), a_right(a)),
    numpy.multiply([float(dt)/(float(dX))]+
    [float(dt)/(float(2.*dX)) for j in range(1,J-1)]+
    [float(dt)/(float(dX))],
    numpy.divide(U,g))))
    f_vec_v = lambda U, V, g, a: numpy.subtract(numpy.multiply(-1., f_vec(U, V)),
    numpy.multiply(numpy.subtract(a_left(a), a_right(a)),
    numpy.multiply([float(dt)/(float(dX))]+
    [float(dt)/(float(2.*dX)) for j in range(1,J-1)]+
    [float(dt)/(float(dX))],
    numpy.divide(V,g))))


However, applying our finitie differences stencil and boundary conditions
rigorously to $a_X$ we
would rather do the following:


    f_vec_u = lambda U, V, g, a: numpy.subtract(f_vec(U, V),
                                             numpy.multiply(numpy.subtract(a_left(a), a_right(a)),
                                                            numpy.multiply(float(dt)/(float(2.*dX)),
                                                                           numpy.divide(U,g))))
    
    f_vec_v = lambda U, V, g, a: numpy.subtract(numpy.multiply(-1., f_vec(U, V)),
                                             numpy.multiply(numpy.subtract(a_left(a), a_right(a)),
                                                            numpy.multiply(float(dt)/(float(2.*dX)),
                                                                           numpy.divide(V,g))))

As opposed to [our previous
derivation](http://georg.io/2013/12/Crank_Nicolson_Convection_Diffusion/),
the sigma constants that we fill the non-zero elements of our tridiagonal
matrices with are
functions of $g$:


    sigma_u_func = lambda g: numpy.divide(float(D_u*dt)/float(2.*dX*dX),
                                          numpy.multiply(g, g))
    sigma_v_func = lambda g: numpy.divide(float(D_v*dt)/float(2.*dX*dX),
                                          numpy.multiply(g, g))

To approximate $g_X$ (thus $G_{XX}$ before our change of notation), we follow
exactly the same
technique and train of thought as above for $a_X$.


    # syntax to get one element in an array: http://stackoverflow.com/a/7332880/3078529
    g_left = lambda g: numpy.concatenate((g[1:J], g[J-1:J]))
    g_right = lambda g: numpy.concatenate((g[0:1], g[0:J-1]))

As for our approximation of $a_X$, I would be inclined to divide by $\Delta X$
rather than $2 \Delta X$
at the boundaries:

    rho_u_func = lambda g, a: numpy.multiply(float(-dt)/float(4.*dX),
                                      numpy.add(numpy.divide(a, g),             
    numpy.multiply(numpy.subtract(g_left(g), g_right(g)),
    numpy.divide([float(D_u)/dX]+
    [float(D_u)/(2.*dX) for j in range(1,J-1)]+
    [float(D_u)/dX],
    numpy.power(g, 3)))))
    rho_v_func = lambda g, a: numpy.multiply(float(-dt)/float(4.*dX),
                                      numpy.add(numpy.divide(a, g),
    numpy.multiply(numpy.subtract(g_left(g), g_right(g)),
    numpy.divide([float(D_v)/dX]+
    [float(D_v)/(2.*dX) for j in range(1,J-1)]+
    [float(D_v)/dX],
    numpy.power(g, 3)))))

However, for the time being we will apply our stencil and boundary conditions
without modification.

This concludes our derivation of [constants
$\rho$](http://georg.io/2013/12/Crank_Nicolson_Convection_Diffusion/)
that are now functions of $g_X$ and $a$.


    rho_u_func = lambda g, a: numpy.multiply(float(-dt)/float(4.*dX),
                                          numpy.add(numpy.divide(a, g),
                                                    numpy.multiply(numpy.subtract(g_left(g), g_right(g)),
                                                                   numpy.divide(float(D_u)/(2.*dX),
                                                                                numpy.power(g, 3)))))
    rho_v_func = lambda g, a: numpy.multiply(float(-dt)/float(4.*dX),
                                          numpy.add(numpy.divide(a, g),
                                                    numpy.multiply(numpy.subtract(g_left(g), g_right(g)),
                                                                   numpy.divide(float(D_v)/(2.*dX),
                                                                                numpy.power(g, 3)))))

To approximate $g_t$ and using our assumption from above, $G_{Xt} = G_{tX}$ the
time derivative
of $g$ is given by:

$$\frac{\partial g}{\partial t} = \frac{\partial a(X,t)}{\partial X}.$$

We use the exact same finite differences stencil as we already did for $a_X$
above.

However, I noticed that I gain some numerical stability when integrating $g$
if I divide by $\Delta X$ rather tha $2 \Delta X$ at the boundaries.

This is just an observation that I made but I have not yet thought about the
implications of this properly.


    g_rhs = lambda t, g, a: numpy.divide(numpy.subtract(a_left(a), a_right(a)),
                                         numpy.array([dX]+[2.*dX for j in range(J-2)]+[dX]))

We decided to model the space grid in `x_grid` so that the grid points `X_grid`
of the variable grid (on which variables `U`, `V`, `g`, and `a` live)
fall into the center between two consecutive space grid points.

Since the local velocities of the space grid points in `x_grid` are governed by
variables that
live `dX` far away in positions `X_grid`, we need to approximate how fast the
space grid points move.
Away from the boundaries ($j=1,\ldots,J-2$) we approximate the velocity of the
space grid points by
the average of the two adjacent `a` values.
At the boundaries we approximate this velocity by a single value. We note
however, that this approximation assigns a non-zero velocity
to the space grid point that represents the origin.
Using our boundary condition $G(0,t) = 0$, the velocity of the space grid point
that represents the origin is
set equal to zero.


    x_grid_rhs = lambda t, x_grid, a: numpy.concatenate(([0.],
                                                         numpy.divide(numpy.add(a[0:J-1], a[1:J]), 2.),
                                                         a[J-1:J]))

To integrate `g` and `x_grid` we use
[`scipy.integrate.ode`](http://docs.scipy.org/doc/scipy/reference/generated/scip
y.integrate.ode.html).


    stepper = integrate.ode(g_rhs)
    stepper = stepper.set_integrator('dopri5', nsteps=10, max_step=dt)
    ti = 0
    stepper = stepper.set_initial_value(g, ti*dt)
    stepper = stepper.set_f_params(a)


    x_grid_ode = integrate.ode(x_grid_rhs)
    x_grid_ode = x_grid_ode.set_integrator('dopri5', nsteps=10, max_step=dt)
    ti = 0
    x_grid_ode = x_grid_ode.set_initial_value(x_grid, ti*dt)
    x_grid_ode = x_grid_ode.set_f_params(a)

As we iterate over time, we will store the values of our variables for later
plotting and analysis.


    U_record = []
    V_record = []
    g_record = []
    a_record = []
    x_record = []
    
    U_record.append(U)
    V_record.append(V)
    g_record.append(g)
    a_record.append(numpy.array(a(U)))
    x_record.append(x_grid)

Let us iterate over time now:


    for ti in range(1,N):
        sigma_u = sigma_u_func(g)
        sigma_v = sigma_v_func(g)
        rho_u = rho_u_func(g, a(U))
        rho_v = rho_v_func(g, a(U))
        
        A_u = numpy.diagflat([-sigma_u[j]+rho_u[j] for j in range(1,J)], -1) +\
              numpy.diagflat([1.+sigma_u[0]+rho_u[0]]+[1.+2.*sigma_u[j] for j in range(1,J-1)]+[1.+sigma_u[J-1]-rho_u[J-1]]) +\
              numpy.diagflat([-(sigma_u[j]+rho_u[j]) for j in range(0,J-1)], 1)
            
        B_u = numpy.diagflat([sigma_u[j]-rho_u[j] for j in range(1,J)], -1) +\
              numpy.diagflat([1.-sigma_u[0]-rho_u[0]]+[1.-2.*sigma_u[j] for j in range(1,J-1)]+[1.-sigma_u[J-1]+rho_u[J-1]]) +\
              numpy.diagflat([sigma_u[j]+rho_u[j] for j in range(0,J-1)], 1)
            
        A_v = numpy.diagflat([-sigma_v[j]+rho_v[j] for j in range(1,J)], -1) +\
              numpy.diagflat([1.+sigma_v[0]+rho_v[0]]+[1.+2.*sigma_v[j] for j in range(1,J-1)]+[1.+sigma_v[J-1]-rho_v[J-1]]) +\
              numpy.diagflat([-(sigma_v[j]+rho_v[j]) for j in range(0,J-1)], 1)
            
        B_v = numpy.diagflat([sigma_v[j]-rho_v[j] for j in range(1,J)], -1) +\
              numpy.diagflat([1.-sigma_v[0]-rho_v[0]]+[1.-2.*sigma_v[j] for j in range(1,J-1)]+[1.-sigma_v[J-1]+rho_v[J-1]]) +\
              numpy.diagflat([sigma_v[j]+rho_v[j] for j in range(0,J-1)], 1)
                
        U_new = numpy.linalg.solve(A_u, B_u.dot(U) + f_vec_u(U, V, g, a(U)))
        V_new = numpy.linalg.solve(A_v, B_v.dot(V) + f_vec_v(U, V, g, a(U)))
        
        while stepper.successful() and stepper.t + dt < ti*dt:
            stepper.integrate(stepper.t + dt)
        while x_grid_ode.successful() and x_grid_ode.t + dt < ti*dt:
            x_grid_ode.integrate(x_grid_ode.t + dt)
            
        # these are the correct "y" values to save for the current time step since
        # we integrate only up to t < ti*dt
        g_new = stepper.y
        x_grid_new = x_grid_ode.y
        
        stepper = stepper.set_f_params(a(U))
        x_grid_ode = x_grid_ode.set_f_params(a(U))
        
        U = U_new
        V = V_new
        g = g_new
        x_grid = x_grid_new
        
        U_record.append(U)
        V_record.append(V)
        g_record.append(g)
        a_record.append(numpy.array(a(U)))
        x_record.append(x_grid)

To get some validation of our numerical integration we make use of the fact that
protein mass
should be conserved and equal `total_protein` throughout our simulations.

When we work out the total protein mass in our domain, we need to do so in space
coordinates rather than
material coordinates.
Here, concentrations are equivalent to the amount of protein per unit of (space)
length.

Since we track the endpoints `x_grid` of our `J` intervals, we can easily work
out how wide
each interval is from `x_grid` and with
[`numpy.diff`](http://docs.scipy.org/doc/numpy/reference/generated/numpy.diff.ht
ml).

Our initial conditions were chosen so that the total protein mass equals `2.26`:


    sum(numpy.multiply(numpy.diff(x_record[0]),U_record[0]) + numpy.multiply(numpy.diff(x_record[0]),V_record[0]))

    2.2599999999999962



At the end of our integration, the total protein mass is still relatively close
to that initial
figure - hence we can have some confidence in the accuracy of our results:


    sum(numpy.multiply(numpy.diff(x_record[-1]),U_record[-1]) + numpy.multiply(numpy.diff(x_record[-1]),V_record[-1]))

    2.2244799098191379



Let us now visualzie the concetration profile of `U` in space and over time.

I have not yet figured out how to visualzie `U` in space coordinates `x_grid`
and,
for the time being, will plot `U` against material coordinates `X_grid`.


    U_record = numpy.array(U_record)
    V_record = numpy.array(V_record)
    
    fig, ax = pyplot.subplots()
    pyplot.xlabel('X'); pyplot.ylabel('t')
    heatmap = ax.pcolormesh(x_record[0], t_grid, U_record, vmin=0., vmax=1.2)
    colorbar = pyplot.colorbar(heatmap)
    colorbar.set_label('concentration U')


![png]({{site.imgbase}}2013-12-05-Reaction_Diffusion_Growing_Domain_files/2013-12-05-Reaction_Diffusion_Growing_Domain_64_0.png)


As we can see, polarity is lost contrary to what happens on
[a domain of fixed size under the same
conditions](http://georg.io/2013/12/Crank_Nicolson/).

Since we also know that the total amount of protein remains roughly constants,
we can conclude
that the total concetration (total protein mass per domain length) is important:
if the total concentration falls below a certain value then polarity is lost.

This effect has been explained by
[Walther *et
al.*](http://link.springer.com/article/10.1007%2Fs11538-012-9766-5).

Let us also take a look at how the local growth velocity `a` behaves over time:


    a_record = numpy.array(a_record)
    
    fig, ax = pyplot.subplots()
    pyplot.xlabel('X'); pyplot.ylabel('t')
    heatmap = ax.pcolormesh(X_grid, t_grid, a_record)
    colorbar = pyplot.colorbar(heatmap)
    colorbar.set_label('local growth velocity a(X,t)')


![png]({{site.imgbase}}/2013-12-05-Reaction_Diffusion_Growing_Domain_files/2013-12-05-Reaction_Diffusion_Growing_Domain_67_0.png)


We can see that the local growth velocity of the volume elements $X$ shadows the
concentration of `U` - just as we defined above.


    max(a_record[-1])

    0.00017910058794305358



The growth local growth velocity `a` does however not drop to zero since `U`
also does not drop to zero
towards the end.
