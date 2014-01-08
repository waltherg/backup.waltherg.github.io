---
layout: default
date: 2014-01-08
title: "Reaction Diffusion System on an Accelerating Domain"
tags: Python Numpy Numerical-Analysis Partial-Differential-Equation
---

# Reaction Diffusion System on an Accelerating Domain

In a [previous blog
post](http://georg.io/2013/12/Reaction_Diffusion_Growing_Domain) we derived
equations
that describe the time behaviour of a reaction-diffusion system on a growing
space domain.

In that work we assumed that the velocity of domain growth is an increasing
function of
one of the two unknown variables (here, protein concentrations) described by our
reaction diffusion system.

Let us add another layer to this problem and assume that **not the growth
velocity** is
dependent on the unknown protein concentration but that **the growth
acceleration** is
a function of this unknown concentration.

We derived the equations that describe the time dynamics of our previous
(velocity) problem
in material coordinates
[here](http://georg.io/2013/12/Reaction_Diffusion_Growing_Domain#specification_o
f_the_local_growth_velocity)
and reproduce them for clarity:

$$\frac{\partial u}{\partial t} = \frac{D_u}{g^2} \frac{\partial^2 u}{\partial
X^2} - \left( D_u \frac{g_X}{g^3} + \frac{a}{g} \right) \frac{\partial
u}{\partial X} + f(u,v) - \frac{a_X}{g} u,$$

$$\frac{\partial v}{\partial t} = \frac{D_v}{g^2} \frac{\partial^2 v}{\partial
X^2} - \left( D_v \frac{g_X}{g^3} + \frac{a}{g} \right) \frac{\partial
v}{\partial X} - f(u,v) - \frac{a_X}{g} v,$$

$$\frac{\partial g}{\partial t} = \frac{\partial a}{\partial X},$$

taken with initial and boundary conditions for $u$, $v$, and $g$ as
[described earlier](http://georg.io/2013/12/Reaction_Diffusion_Growing_Domain).

Let us now introduce a fourth equation to this system to describe the time
behaviour of
the growth velocity $a$ as a function of its acceleration:

$$\frac{\partial a}{\partial t} = s(X,t),$$

where $s(X,t)$ is the local growth acceleration.

In this expanded system of partial differential equations the time behaviour of
the, now, unknown variable $a$
is described by an uncoupled partial differential equation.
Since there is no spatial coupling in this partial differential equation, we do
not need to impose
any boundary conditions.
As initial condition, we will assume that the system is at rest at first:

$$a(X,0) = 0.$$

Let us now modify our
[previous
code](http://georg.io/2013/12/Reaction_Diffusion_Growing_Domain#python_code)
to simulate this expanded system.

Most of this code is just a copy-and-paste of our
[previous
code](http://georg.io/2013/12/Reaction_Diffusion_Growing_Domain#python_code)
so please refer to our comments there if anything is unclear.

    %matplotlib inline

    import numpy
    from scipy import integrate
    from matplotlib import pyplot
    numpy.set_printoptions(precision=3)

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

    k0 = 0.067
    f_vec = lambda U, V: numpy.multiply(dt, numpy.subtract(numpy.multiply(V, 
                         numpy.add(k0, numpy.divide(numpy.multiply(U,U), numpy.add(1., numpy.multiply(U,U))))), U))

The initial condition for the growth velocity `a` is set to zero everywhere so
that the space domain is at rest at first.

The expression for the growth acceleration `s` is the exact same expression we
used
[earlier](http://georg.io/2013/12/Reaction_Diffusion_Growing_Domain#python_code)
for the growth velocity.

    total_protein = 2.26
    
    no_high = 10
    U = numpy.array([0.1 for i in range(no_high,J)] + [2. for i in range(0,no_high)])
    V = numpy.array([float(total_protein-dX*sum(U))/float(J*dX) for i in range(0,J)])
    g = numpy.array([1. for j in range(J)])
    a = numpy.array([0. for j in range(J)])
    s = lambda U: numpy.array([0.001*X_grid[j]*U[j] for j in range(J)])

This is the *polarized* initial condition we choose for our concentration system
`U` and `V`:

    pyplot.ylim((0., 2.1))
    pyplot.xlabel('X'); pyplot.ylabel('concentration')
    pyplot.plot(X_grid, U)
    pyplot.plot(X_grid, V)
    pyplot.show()

![png](https://dl.dropboxusercontent.com/u/129945779/georgio/2014-01-08-Reaction_Diffusion_Accelerating_Domain_files/2014-01-08-Reaction_Diffusion_Accelerating_Domain_10_0.png)

As before, the initial condition for the slope `g` of the trajectories $G$ is
`1.` everywhere:

    pyplot.plot(X_grid, g)
    pyplot.xlabel('X'); pyplot.ylabel('g')
    pyplot.show()

![png](https://dl.dropboxusercontent.com/u/129945779/georgio/2014-01-08-Reaction_Diffusion_Accelerating_Domain_files/2014-01-08-Reaction_Diffusion_Accelerating_Domain_12_0.png)

Initially, `s` looks the same as `a` did in our [previous
code](http://georg.io/2013/12/Reaction_Diffusion_Growing_Domain#python_code),
while in this version of our code `a` is set initially to `0.` everywhere:

    pyplot.plot(X_grid, a)
    pyplot.plot(X_grid, s(U))
    pyplot.xlabel('X'); pyplot.ylabel('g')
    pyplot.ylim((-.0001, 0.0021))
    pyplot.show()

![png](https://dl.dropboxusercontent.com/u/129945779/georgio/2014-01-08-Reaction_Diffusion_Accelerating_Domain_files/2014-01-08-Reaction_Diffusion_Accelerating_Domain_14_0.png)

    # syntax to get one element in an array: http://stackoverflow.com/a/7332880/3078529
    a_left = lambda a: numpy.concatenate((a[1:J], a[J-1:J]))
    a_right = lambda a: numpy.concatenate((a[0:1], a[0:J-1]))

    f_vec_u = lambda U, V, g, a: numpy.subtract(f_vec(U, V),
                                             numpy.multiply(numpy.subtract(a_left(a), a_right(a)),
                                                            numpy.multiply(float(dt)/(float(2.*dX)),
                                                                           numpy.divide(U,g))))
    
    f_vec_v = lambda U, V, g, a: numpy.subtract(numpy.multiply(-1., f_vec(U, V)),
                                             numpy.multiply(numpy.subtract(a_left(a), a_right(a)),
                                                            numpy.multiply(float(dt)/(float(2.*dX)),
                                                                           numpy.divide(V,g))))

    sigma_u_func = lambda g: numpy.divide(float(D_u*dt)/float(2.*dX*dX),
                                          numpy.multiply(g, g))
    sigma_v_func = lambda g: numpy.divide(float(D_v*dt)/float(2.*dX*dX),
                                          numpy.multiply(g, g))

    # syntax to get one element in an array: http://stackoverflow.com/a/7332880/3078529
    g_left = lambda g: numpy.concatenate((g[1:J], g[J-1:J]))
    g_right = lambda g: numpy.concatenate((g[0:1], g[0:J-1]))

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

Note that here we use slightly more meaningful variable names for the ODE
steppers for `g` and `x_grid` than
in our [previous
code](http://georg.io/2013/12/Reaction_Diffusion_Growing_Domain#python_code).

The ODE stepper for `a` is of the same form except that the right-hand side is
governed by the
growth acceleration `s`:

    g_rhs = lambda t, g, a: numpy.divide(numpy.subtract(a_left(a), a_right(a)),
                                         numpy.array([dX]+[2.*dX for j in range(J-2)]+[dX]))
    
    g_stepper = integrate.ode(g_rhs)
    g_stepper = g_stepper.set_integrator('dopri5', nsteps=10, max_step=dt)
    g_stepper = g_stepper.set_initial_value(g, 0.)
    g_stepper = g_stepper.set_f_params(a)

    a_rhs = lambda t, a, s: s
    
    a_stepper = integrate.ode(a_rhs)
    a_stepper = a_stepper.set_integrator('dopri5', nsteps=10, max_step=dt)
    a_stepper = a_stepper.set_initial_value(a, 0.)
    a_stepper = a_stepper.set_f_params(s(U))

    x_grid_rhs = lambda t, x_grid, a: numpy.concatenate(([0.],
                                                         numpy.divide(numpy.add(a[0:J-1], a[1:J]), 2.),
                                                         a[J-1:J]))
    
    x_stepper = integrate.ode(x_grid_rhs)
    x_stepper = x_stepper.set_integrator('dopri5', nsteps=10, max_step=dt)
    x_stepper = x_stepper.set_initial_value(x_grid, 0.)
    x_stepper = x_stepper.set_f_params(a)

    U_record = []
    V_record = []
    g_record = []
    a_record = []
    x_record = []
    
    U_record.append(U)
    V_record.append(V)
    g_record.append(g)
    a_record.append(a)
    x_record.append(x_grid)

Let us now integrate this system for a total of `N` time points.

Notice that we update the ODE stepper of `g`, `a` and `x_grid` every time step
with the current state
of their corresponding right-hand-side expressions.
This is done with a call to `.set_f_params()` on the corresponding objects.

    for ti in range(1,N):
        sigma_u = sigma_u_func(g)
        sigma_v = sigma_v_func(g)
        rho_u = rho_u_func(g, a)
        rho_v = rho_v_func(g, a)
        
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
                
        U_new = numpy.linalg.solve(A_u, B_u.dot(U) + f_vec_u(U, V, g, a))
        V_new = numpy.linalg.solve(A_v, B_v.dot(V) + f_vec_v(U, V, g, a))
        
        while a_stepper.successful() and a_stepper.t + dt < ti*dt:
            a_stepper.integrate(a_stepper.t + dt)
        while g_stepper.successful() and g_stepper.t + dt < ti*dt:
            g_stepper.integrate(g_stepper.t + dt)
        while x_stepper.successful() and x_stepper.t + dt < ti*dt:
            x_stepper.integrate(x_stepper.t + dt)
            
        g_stepper = g_stepper.set_f_params(a)
        x_stepper = x_stepper.set_f_params(a)
        a_stepper = a_stepper.set_f_params(s(U))
        
        U = U_new
        V = V_new
        
        # these are the correct "y" values to save for the current time step since
        # we integrate only up to t < ti*dt
        g = g_stepper.y
        a = a_stepper.y
        x_grid = x_stepper.y
        
        U_record.append(U)
        V_record.append(V)
        g_record.append(g)
        a_record.append(a)
        x_record.append(x_grid)

As in our
[previous
code](http://georg.io/2013/12/Reaction_Diffusion_Growing_Domain#python_code)
protein mass is conserved by definition of our boundary conditions for `U` and
`V`.

To make certain that our numerical integration does conserve protein mass (at
least to
some sensible degree) we work out the initial total mass and final total mass
respectively:

    sum(numpy.multiply(numpy.diff(x_record[0]),U_record[0]) + numpy.multiply(numpy.diff(x_record[0]),V_record[0]))

    2.2599999999999962

    sum(numpy.multiply(numpy.diff(x_record[-1]),U_record[-1]) + numpy.multiply(numpy.diff(x_record[-1]),V_record[-1]))

    2.2543232809714371

These numbers look good and we therefore assume that our numerical integration
is somewhat stable.

A look at a kymograph of the concentration of `U` reveals that the initial
condition on `U` is
lost quickly - which will be due to reasons we discussed
[earlier](http://georg.io/2013/12/Reaction_Diffusion_Growing_Domain).

    U_record = numpy.array(U_record)
    V_record = numpy.array(V_record)
    
    fig, ax = pyplot.subplots()
    pyplot.xlabel('X'); pyplot.ylabel('t')
    heatmap = ax.pcolormesh(x_record[0], t_grid, U_record, vmin=0., vmax=1.2)
    colorbar = pyplot.colorbar(heatmap)
    colorbar.set_label('concentration U')

![png](https://dl.dropboxusercontent.com/u/129945779/georgio/2014-01-08-Reaction_Diffusion_Accelerating_Domain_files/2014-01-08-Reaction_Diffusion_Accelerating_Domain_32_0.png)

Since with the above setup **we only introduce acceleration and no
deceleration**, we are not surprised
to observe that the growth velocity simply increases over time:

    a_record = numpy.array(a_record)
    
    fig, ax = pyplot.subplots()
    pyplot.xlabel('X'); pyplot.ylabel('t')
    heatmap = ax.pcolormesh(X_grid, t_grid, a_record)
    colorbar = pyplot.colorbar(heatmap)
    colorbar.set_label('local growth velocity a(X,t)')

![png](https://dl.dropboxusercontent.com/u/129945779/georgio/2014-01-08-Reaction_Diffusion_Accelerating_Domain_files/2014-01-08-Reaction_Diffusion_Accelerating_Domain_34_0.png)

This continual increase in growth velocity is only due to an initial
acceleration due to the initial condition we imposed on `U`:

    fig, ax = pyplot.subplots()
    pyplot.xlabel('X'); pyplot.ylabel('t')
    s_record = numpy.empty(shape=(N, J))
    for ti in range(N):
        s_record[ti] = s(U_record[ti])
    heatmap = ax.pcolormesh(X_grid, t_grid, s_record)
    colorbar = pyplot.colorbar(heatmap)
    colorbar.set_label('local growth acceleration s')

![png](https://dl.dropboxusercontent.com/u/129945779/georgio/2014-01-08-Reaction_Diffusion_Accelerating_Domain_files/2014-01-08-Reaction_Diffusion_Accelerating_Domain_36_0.png)
