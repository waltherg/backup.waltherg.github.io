---
layout: default
title: "Crank-Nicolson with Variable Diffusivity"
date: 2013-12-04
tags: Python Numpy Numerical-Analysis Partial-Differential-Equation
---

# Crank-Nicolson with Variable Diffusivity

We [have implemented](http://georg.io/2013/12/03/Crank_Nicolson.html#a_cranknico
lson_example_in_python)
the Crank-Nicolson (CN) method for a two-variable reaction-diffusion system with
constant grid parameters, $\Delta t$ and $\Delta x$, and system parameters
(including diffusion coefficients $D_u$ and $D_v$).

When we keep all of these parameters constant, we can define constants $\sigma_u
= \frac{D_u \Delta t}{2 \Delta x^2}$
and $\sigma_v = \frac{D_v \Delta t}{2 \Delta x^2}$ that
define the non-zero entries of
[the two tridiagonal matrices](http://georg.io/2013/12/03/Crank_Nicolson.html#re
ordering_stencil_into_linear_system) defined by
[the CN stencil](http://georg.io/2013/12/03/Crank_Nicolson.html#the_cranknicolso
n_stencil).

To reuse [our previous implementation of the CN method](http://georg.io/2013/12/
03/Crank_Nicolson.html#a_cranknicolson_example_in_python)
when diffusion coefficients $D_u$ and $D_v$ are functions of time $t$ (the
properties of the material that the
[molecular species $u$ and $v$](http://georg.io/2013/12/03/Crank_Nicolson.html)
diffuse in change)
we can use [Python `lambda` functions](http://www.diveintopython.net/power_of_in
trospection/lambda_functions.html).

Suppose that

$$D_u(t) = (1 + 0.05 t) D_{u,0},$$

$$D_v(t) = (1 + 0.05 t) D_{v,0},$$

where $D_{u,0}$ and $D_{v,0}$ are the constant diffusion coefficients
[we used previously](http://georg.io/2013/12/03/Crank_Nicolson.html#specify_syst
em_parameters_and_the_reaction_term).

Then in our Python code, `sigma_u` and `sigma_v` become

    sigma_u = lambda t: float((1.+0.05*t)*D_u*dt)/float(2.*dx*dx)
    sigma_v = lambda t: float((1.+0.05*t)*D_v*dt)/float(2.*dx*dx)

We also need to turn matrices `A_u`, `B_u`, `A_v`, and `B_v` into `lambda`
functions (see below).

[Reusing our previous code](http://georg.io/2013/12/03/Crank_Nicolson.html#a_cra
nknicolson_example_in_python) with these
modifications looks as follows - with a kymograph of our numerical results and
the time behaviour of `sigma_u` shown below.


    import numpy
    from matplotlib import pyplot
    numpy.set_printoptions(precision=3)
    
    L = 1.
    J = 100
    dx = float(L)/float(J-1)
    x_grid = numpy.array([j*dx for j in range(J)])
    
    T = 200
    N = 1000
    dt = float(T)/float(N-1)
    t_grid = numpy.array([n*dt for n in range(N)])
    
    D_v = float(10.)/float(100.)
    D_u = 0.01 * D_v
    
    k0 = 0.067
    f_vec = lambda U, V: numpy.multiply(dt, numpy.subtract(numpy.multiply(V, 
                         numpy.add(k0, numpy.divide(numpy.multiply(U,U), numpy.add(1., numpy.multiply(U,U))))), U))
    g = lambda u, v: -f(u,v)
     
    sigma_u = lambda t: float((1.+0.05*t)*D_u*dt)/float(2.*dx*dx)
    sigma_v = lambda t: float((1.+0.05*t)*D_v*dt)/float(2.*dx*dx)
    
    total_protein = 2.26
    
    no_high = 10
    U = numpy.array([0.1 for i in range(no_high,J)] + [2. for i in range(0,no_high)])
    V = numpy.array([float(total_protein-dx*sum(U))/float(J*dx) for i in range(0,J)])
    
    A_u = lambda t: numpy.diagflat([-sigma_u(t) for i in range(J-1)], -1) +\
          numpy.diagflat([1.+sigma_u(t)]+[1.+2.*sigma_u(t) for i in range(J-2)]+[1.+sigma_u(t)]) +\
          numpy.diagflat([-sigma_u(t) for i in range(J-1)], 1)
            
    B_u = lambda t: numpy.diagflat([sigma_u(t) for i in range(J-1)], -1) +\
          numpy.diagflat([1.-sigma_u(t)]+[1.-2.*sigma_u(t) for i in range(J-2)]+[1.-sigma_u(t)]) +\
          numpy.diagflat([sigma_u(t) for i in range(J-1)], 1)
            
    A_v = lambda t: numpy.diagflat([-sigma_v(t) for i in range(J-1)], -1) +\
          numpy.diagflat([1.+sigma_v(t)]+[1.+2.*sigma_v(t) for i in range(J-2)]+[1.+sigma_v(t)]) +\
          numpy.diagflat([-sigma_v(t) for i in range(J-1)], 1)
            
    B_v = lambda t: numpy.diagflat([sigma_v(t) for i in range(J-1)], -1) +\
          numpy.diagflat([1.-sigma_v(t)]+[1.-2.*sigma_v(t) for i in range(J-2)]+[1.-sigma_v(t)]) +\
          numpy.diagflat([sigma_v(t) for i in range(J-1)], 1)
            
    U_record = []
    V_record = []
    
    U_record.append(U)
    V_record.append(V)
    
    for ti in range(1,N):
        t = ti*dt
        
        U_new = numpy.linalg.solve(A_u(t), B_u(t).dot(U) + f_vec(U,V))
        V_new = numpy.linalg.solve(A_v(t), B_v(t).dot(V) - f_vec(U,V))
        
        U = U_new
        V = V_new
        
        U_record.append(U)
        V_record.append(V)


    U_record = numpy.array(U_record)
    V_record = numpy.array(V_record)
    
    fig = pyplot.figure()
    fig.set_size_inches(10.5,6.5)
    ax1 = fig.add_subplot(122)
    ax1.set_xlabel('x')
    ax1.set_ylim(T)
    heatmap = ax1.pcolor(x_grid, t_grid, U_record, vmin=0., vmax=1.2)
    colorbar = pyplot.colorbar(heatmap)
    colorbar.set_label('concentration U')
    ax2 = fig.add_subplot(121, sharey=ax1)
    ax2.set_xlabel('sigma_u')
    ax2.set_ylabel('t')
    ax2.set_xlim([sigma_u(0), sigma_u(T)])
    line = ax2.plot([sigma_u(t) for t in t_grid], t_grid)


![png](https://dl.dropboxusercontent.com/u/129945779/georgio/2013-12-04-Crank_Nicolson_Variable_Diffusivity_files/2013-12-04-Crank_Nicolson_Variable_Diffusivity_3_0.png)

