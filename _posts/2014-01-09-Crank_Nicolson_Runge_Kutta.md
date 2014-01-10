---
layout: default
title: "Combining Crank-Nicolson and Runge-Kutta to Solve a Reaction-Diffusion System"
date: 2014-01-09
tags: Python Numpy Numerical-Analysis Partial-Differential-Equation
---

# Combining Crank-Nicolson and Runge-Kutta to Solve a Reaction-Diffusion System

We have already [derived](http://georg.io/2013/12/Crank_Nicolson/) the Crank-
Nicolson method
to integrate the following reaction-diffusion system numerically:

$$\frac{\partial u}{\partial t} = D \frac{\partial^2 u}{\partial x^2} + f(u),$$

$$\frac{\partial u}{\partial x}\Bigg|_{x = 0, L} = 0.$$

Please refer to the [earlier blog post](http://georg.io/2013/12/Crank_Nicolson/)
for details.

In our previous derivation, we constructed the following stencil that we would
go on to
rearrange into a system of linear equations that we needed to solve every time
step:

$$\frac{U_j^{n+1} - U_j^n}{\Delta t} = \frac{D}{2 \Delta x^2} \left( U_{j+1}^n -
2 U_j^n + U_{j-1}^n + U_{j+1}^{n+1} - 2 U_j^{n+1} + U_{j-1}^{n+1}\right) +
f(U_j^n),$$

where $j$ and $n$ are space and time grid points respectively.

Rearranging the above set of equations, we effectively integrate the reaction
part with the
[explicit Euler method](https://en.wikipedia.org/wiki/Euler_method) like so:

$$U_j^{n+1} = U_j^n + \Delta t f(U_j^n).$$

For functions $f$ that change rapidly for small changes in their input
([stiff equations](https://en.wikipedia.org/wiki/Stiff_equation)), using
the explicit Euler method may pose stability problems unless we choose a
sufficiently
small $\Delta t$.

Therefore, I have been wondering if it would be possible to use a more
sophisticated
and stable numerical scheme to integrate the reaction part in the context of our
Crank-Nicolson scheme.

For instance, to integrate the reaction part with the
[classical Runge-Kutta method](https://en.wikipedia.org/wiki/Runge%E2%80%93Kutta
_methods#The_Runge.E2.80.93Kutta_method),
we would write out the following set of equations instead of the aforementioned
one:

$$\frac{U_j^{n+1} - U_j^n}{\Delta t} = \frac{D}{2 \Delta x^2} \left( U_{j+1}^n -
2 U_j^n + U_{j-1}^n + U_{j+1}^{n+1} - 2 U_j^{n+1} + U_{j-1}^{n+1}\right) +
\frac{1}{6} \left(k_1 + 2 k_2 + 2 k_3 + k_4 \right),$$

where

$$k_1 = f(U_j^n),$$

$$k_2 = f\left( U_j^n + \frac{\Delta t}{2} k_1 \right),$$

$$k_3 = f\left( U_j^n + \frac{\Delta t}{2} k_2 \right),$$

$$k_4 = f\left( U_j^n + \Delta t k_3 \right).$$

Whether or not doing this makes sense theoretically I am not certain. But going
ahead and implementing this
to the numerical example we discussed
[earlier](http://georg.io/2013/12/Crank_Nicolson) seems to suggest
that this does work.

In the following Python code that is mostly a copy of [our previous
code](http://georg.io/2013/12/Crank_Nicolson/)
we compare the time behaviour and accuracy (measured by mass conservation as our
reaction diffusion system
preserves mass) of the explicit Euler and Runge-Kutta 4 reaction integration.

We realize that the differences between the obtained numerical results are
negligible and we shall
compare both approaches with a stiffer reaction term another time.

We shall also take a look at more sophisticated measures of numerical stability
another time.


    %matplotlib inline
    
    import numpy
    from matplotlib import pyplot


    numpy.set_printoptions(precision=3)


    L = 1.
    J = 200
    dx = float(L)/float(J-1)
    x_grid = numpy.array([j*dx for j in range(J)])


    T = 500
    N = 1000
    dt = float(T)/float(N-1)
    t_grid = numpy.array([n*dt for n in range(N)])


    D_v = float(10.)/float(100.)
    D_u = 0.01 * D_v
    
    k0 = 0.067
    f = lambda u, v: dt*(v*(k0 + float(u*u)/float(1. + u*u)) - u)
    g = lambda u, v: -f(u,v)
     
    sigma_u = float(D_u*dt)/float((2.*dx*dx))
    sigma_v = float(D_v*dt)/float((2.*dx*dx))
    
    total_protein = 2.26


    no_high = 10
    U =  numpy.array([0.1 for i in range(no_high,J)] + [2. for i in range(0,no_high)])
    V = numpy.array([float(total_protein-dx*sum(U))/float(J*dx) for i in range(0,J)])

Let us take a look at the inhomogeneous initial condition:


    pyplot.ylim((0., 2.1))
    pyplot.xlabel('x')
    pyplot.ylabel('concentration')
    pyplot.plot(x_grid, U)
    pyplot.plot(x_grid, V)
    pyplot.show()


![png]({{imgbase}}/2014-01-09-Crank_Nicolson_Runge_Kutta_files/2014-01-09-Crank_Nicolson_Runge_Kutta_9_0.png)


These are the matrices of our system of linear equations whose derivation
[we described earlier](http://georg.io/2013/12/Crank_Nicolson/).


    A_u = numpy.diagflat([-sigma_u for i in range(J-1)], -1) +\
          numpy.diagflat([1.+sigma_u]+[1.+2.*sigma_u for i in range(J-2)]+[1.+sigma_u]) +\
          numpy.diagflat([-sigma_u for i in range(J-1)], 1)
            
    B_u = numpy.diagflat([sigma_u for i in range(J-1)], -1) +\
          numpy.diagflat([1.-sigma_u]+[1.-2.*sigma_u for i in range(J-2)]+[1.-sigma_u]) +\
          numpy.diagflat([sigma_u for i in range(J-1)], 1)
            
    A_v = numpy.diagflat([-sigma_v for i in range(J-1)], -1) +\
          numpy.diagflat([1.+sigma_v]+[1.+2.*sigma_v for i in range(J-2)]+[1.+sigma_v]) +\
          numpy.diagflat([-sigma_v for i in range(J-1)], 1)
            
    B_v = numpy.diagflat([sigma_v for i in range(J-1)], -1) +\
          numpy.diagflat([1.-sigma_v]+[1.-2.*sigma_v for i in range(J-2)]+[1.-sigma_v]) +\
          numpy.diagflat([sigma_v for i in range(J-1)], 1)

Function `f_vec_ee` returns the explicit Euler time step vector while `f_vec_rk`
returns the vector obtained
applying the Runge-Kutta 4 method.


    def f_vec_ee(U,V):
        return numpy.multiply(dt, numpy.subtract(numpy.multiply(V, 
               numpy.add(k0, numpy.divide(numpy.multiply(U,U), numpy.add(1., numpy.multiply(U,U))))), U))


    def f_vec_rk(U, V):
        f_vec = lambda U, V: numpy.subtract(numpy.multiply(V, 
                             numpy.add(k0, numpy.divide(numpy.multiply(U,U), numpy.add(1., numpy.multiply(U,U))))), U)
        k1 = f_vec(U, V)
        k2 = f_vec(U + numpy.multiply(dt/2., k1), V - numpy.multiply(dt/2., k1))
        k3 = f_vec(U + numpy.multiply(dt/2., k2), V - numpy.multiply(dt/2., k2))
        k4 = f_vec(U + numpy.multiply(dt, k3), V - numpy.multiply(dt, k3))
        
        return numpy.multiply(dt/6., k1 + numpy.multiply(2., k2) + numpy.multiply(2., k3) + k4)


    U_record_ee = numpy.empty(shape=(N,J))
    V_record_ee = numpy.empty(shape=(N,J))
    
    U_record_rk = numpy.empty(shape=(N,J))
    V_record_rk = numpy.empty(shape=(N,J))
    
    U_record_ee[0][:] = U[:]
    V_record_ee[0][:] = V[:]
    
    U_record_rk[0][:] = U[:]
    V_record_rk[0][:] = V[:]
    
    for ti in range(1,N):
        U_record_ee[ti][:] = numpy.linalg.solve(A_u, B_u.dot(U_record_ee[ti-1][:]) +
                                                f_vec_ee(U_record_ee[ti-1][:],V_record_ee[ti-1][:]))
        V_record_ee[ti][:] = numpy.linalg.solve(A_v, B_v.dot(V_record_ee[ti-1][:]) - 
                                                f_vec_ee(U_record_ee[ti-1][:],V_record_ee[ti-1][:]))
        
        U_record_rk[ti][:] = numpy.linalg.solve(A_u, B_u.dot(U_record_rk[ti-1][:]) +
                                                f_vec_rk(U_record_rk[ti-1][:],V_record_rk[ti-1][:]))
        V_record_rk[ti][:] = numpy.linalg.solve(A_v, B_v.dot(V_record_rk[ti-1][:]) - 
                                                f_vec_rk(U_record_rk[ti-1][:],V_record_rk[ti-1][:]))

The initial protein mass in our system:


    print 'Explicit Euler', numpy.sum(numpy.multiply(dx, U_record_ee[0]) + numpy.multiply(dx, V_record_ee[0]))
    print 'Runge-Kutta 4 ', numpy.sum(numpy.multiply(dx, U_record_ee[0]) + numpy.multiply(dx, V_record_ee[0]))

    Explicit Euler 2.26
    Runge-Kutta 4  2.26


Since our reaction-diffusion system preserves mass, we should retain the same
protein mass at steady-state
for both numerical approaches:


    print 'Explicit Euler %.14f' % numpy.sum(numpy.multiply(dx, U_record_ee[-1]) + numpy.multiply(dx, V_record_ee[-1]))
    print 'Runge-Kutta 4  %.14f' % numpy.sum(numpy.multiply(dx, U_record_rk[-1]) + numpy.multiply(dx, V_record_rk[-1]))

    Explicit Euler 2.25999999998857
    Runge-Kutta 4  2.25999999998951


We realize that the difference between the two numerical methods is negligible
and we shall
compare both approaches for a stiffer system another time.

A plot of the steady-state concentration profiles confirms that we cannot
observe a significant differences
between the results generated by both methods (varying `J` and `N` paints the
same pictures).


    pyplot.ylim((0., 2.1))
    pyplot.xlabel('x')
    pyplot.ylabel('concentration')
    pyplot.plot(x_grid, U_record_ee[-1])
    pyplot.plot(x_grid, V_record_ee[-1])
    pyplot.plot(x_grid, U_record_rk[-1])
    pyplot.plot(x_grid, V_record_rk[-1])
    pyplot.show()


![png]({{imgbase}}/2014-01-09-Crank_Nicolson_Runge_Kutta_files/2014-01-09-Crank_Nicolson_Runge_Kutta_22_0.png)


Kymograph of `U` integrated with the explicit Euler method.


    fig, ax = pyplot.subplots()
    pyplot.xlabel('x')
    pyplot.ylabel('t')
    pyplot.ylim((0., T))
    heatmap = ax.pcolormesh(x_grid, t_grid, U_record_ee, vmin=0., vmax=1.2)
    colorbar = pyplot.colorbar(heatmap)
    colorbar.set_label('concentration U')


![png]({{imgbase}}/2014-01-09-Crank_Nicolson_Runge_Kutta_files/2014-01-09-Crank_Nicolson_Runge_Kutta_24_0.png)


Kymograph of `U` integrated with the Runge-Kutta 4 method.


    fig, ax = pyplot.subplots()
    pyplot.xlabel('x')
    pyplot.ylabel('t')
    pyplot.ylim((0., T))
    heatmap = ax.pcolormesh(x_grid, t_grid, U_record_rk, vmin=0., vmax=1.2)
    colorbar = pyplot.colorbar(heatmap)
    colorbar.set_label('concentration U')


![png]({{imgbase}}/2014-01-09-Crank_Nicolson_Runge_Kutta_files/2014-01-09-Crank_Nicolson_Runge_Kutta_26_0.png)

