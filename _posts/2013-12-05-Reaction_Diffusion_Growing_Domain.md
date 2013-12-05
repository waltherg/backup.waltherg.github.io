---
layout: default
date: 2013-12-05
title: "Reaction-Diffusion System on a Growing Domain"
tags: wip
---

**This post is work in progress**

# Reaction-Diffusion System on a Growing Domain

We simulate the same two-variable reaction-diffusion system in one space
dimensions
[that we considered previously](http://georg.io/2013/12/03/Crank_Nicolson.html#a_cranknicolson_example_in_python).

Specifically, we are interested in the behaviour of this system when the one-
dimensional domain grows.
Reaction-diffusion systems on growing domains have been discussed extensively by
[Edmund J Crampin](http://solo.bodleian.ox.ac.uk/primo_library/libweb/action/dlDisplay.do?vid=OXVU1&docId=oxfaleph015225500)
and in his related
[publications](http://people.eng.unimelb.edu.au/ecrampin/publications.html).

## Reynolds Transport Theorem

The general form of our reaction-diffusion system subject to flow $a$ of the
particle-carrying liquid
can be derived using the [Reynolds Transport
Theorem](http://en.wikipedia.org/wiki/Reynolds_transport_theorem).

## Trajectories of Volume Elements

Let us label the infinitessimal volume elements of our one-dimensional domain
with $X \in [0, L]$, where $L$ is the length of the domain.

The space coordinate $x$ of the material coordinate $X$ at time $t$ is described
by the trajectory of $X$

$$x = G(X,t),$$

and $G$ is subject to initial condition
$G(X,0) = X$ (the initial position in space of the element labeled with $X$ is
equal to $X$),
and boundary condition
$G(0,t) = 0$ (the origin does not move so that we model growth and not
translation of our domain).

The trajectory of $X$ is related to its initial condition through

$$G(X,t) = G(X,0) + X \int_0^t{a(X,s) ds},$$

where $a(X,t)$ is the flow velocity and describes how fast element $X$ moves at
time $t$.
This expression also enforces the boundary condition in the origin.

## Specification of the Flow Velocity

In the simplest case the flow velocity is a constant $a(X,t) = a_0$ so that the
trajectory becomes
$G(X,t) = X (1 + a_0 t)$.

Generally, however, $a(X,t)$ is unknown so that we can only express $G$ as a
partial differential equation,
which we obtain from the above expression for $G(X,t)$ using the
[fundamental theorem of
calculus](http://en.wikipedia.org/wiki/Fundamental_theorem_of_calculus):

$$\frac{\partial G}{\partial t} = G_t = X a(X,t),$$

with boundary condition $G(0,t) = 0$ and initial condition $G(X,0) = 0$.

## Our Reaction-Diffusion System in Material Coordinates

We substitute the identity $x = G(X,t)$ into the above partial differential
equation and obtain

$$\frac{\partial u}{\partial t} = \frac{D_u}{G_X^2} \frac{\partial^2 u}{\partial
x^2} - \left( \frac{G_{XX}}{G_X^3} + \frac{G_t}{G_X} \right) \frac{\partial
u}{\partial X} + f(u,v) - \frac{G_{Xt}}{G_X} u,$$
