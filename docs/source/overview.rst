Overview
========

Introduction
************

The goal of the ``TPS`` code is to support high-fidelity simulations
of inductively coupled plasma (ICP) torches.  Towards this eventual
goal, the code currently supports single physics flow-only
simulations by solving the compressible Navier--Stokes (NS) equations
and electromagnetic-only simulations by solving the
quasimagnetostatic approximation of Maxwell's equations.  Both models
are discretized using the finite element method (FEM), and both
solvers are implmented using the `MFEM <https://mfem.org>`_ library.
As the project continues, the codebase will evolve to support
additional physical models and multiphysics coupling to enable
efficient, high-fidelity simulations of ICP torches and related plasma
systems.

.. The ``TPS`` code solves the Navier-Stokes (NS) equations or the Maxwell equations for
.. Electro-Magnetics (EM) using the Finite Element Method (FEM). As the project evolves the
.. different equations will be solved simultaneously (fully coupled) so that efficient,
.. high-fidelity simulations of an Inductively Coupled Plasma (ICP) Torch can be made.

.. At the moment, a Discontinuous Galerkin approach has been taken for the solution of the NS
.. equations whereas a Continuous Galerkin (CG) approach has been taken for the solution of the
.. EM equations. These are implemented using the `MFEM <https://mfem.org>`_ library 

*Note that this code is the product of an ongoing research project.
As such, the details described below, including both physical models
and numerical methods are subject to change in future versons of the
software.*

----

Flow Solver
************

Governing Equations
^^^^^^^^^^^^^^^^^^^

We consider the NS equations expressed in conservative form:

:math:`\frac{\partial U}{\partial t}+\nabla\cdot\mathbf{F}\left(U,\nabla V\right)=0`

where :math:`U=\left[\rho,\rho u_i,\rho E\right]^{T}` is the vector of
conservative variables, and :math:`V=\left[\rho,u_i,p\right]^{T}` is
the vector of primitive variables with pressure
:math:`p=\left(\gamma-1\right)\rho\left(E-\frac{1}{2} u_i u_i\right)`
and total energy :math:`E=c_{v}T+\frac{1}{2}\left(u_i u_i\right)`.
Further, the flux in the :math:`i` th direction is given by
:math:`F_i=F^c_i-F^{v}_i` where :math:`F^{c}_{i}, F^{v}_{i}` are the
convective and viscous fluxes in the :math:`i` th direction,
respectively, defined as

:math:`F^{c}_i=\left[\begin{array}{c}
\rho u_i\\
\rho u_i u_j +p \delta_{ij}\\
u_i \left(\rho E+p\right)
\end{array}\right]`

:math:`F^{v}_i=\left[\begin{array}{c}
0 \\
\tau_{ij}\\
u_j \tau_{ji} - q_i
\end{array}\right]`

where :math:`\tau_{ij}=\mu\left(\frac{\partial u_{i}}{\partial
x_{j}}+\frac{\partial u_{j}}{\partial x_{i}}\right)-\frac{2}{3}\mu
\frac{\partial u_k}{\partial k} \delta_{ij}` is the viscous stress
tensor and :math:`q_i=-k \frac{\partial T}{\partial x_i}` is the
viscous heat flux.  The thermal conductivity is given by
:math:`k=\frac{\mu c_{p}}{Pr}` where :math:`Pr` is the Prandtl number,
estimated as :math:`Pr=0.71` for air. The viscosity coefficient
follows Sutherland's law
:math:`\mu=\frac{1.458\cdot10^{-6}T^{3/2}}{T+110.4}`.

These equations may be written as the following first-order system:

:math:`\begin{aligned}\frac{\partial U}{\partial t}+\nabla\cdot\mathbf{F}\left(U,Q\right) & =0\\
Q-\nabla V & =0
\end{aligned}`

----

Numerics
^^^^^^^^

We derive a discontinuous Galerkin formulation of the previous system in the usual manner, 
multiplying by the test function :math:`\ell_{j}\left(\xi\right)` and integrating over 
the element

:math:`\frac{\mathrm{d}}{\mathrm{d}t}\intop_{\Omega_{e}}U^{e}\ell_{j}\left(x\right)\mathrm{d}x-\intop_{\Omega_{e}}\mathbf{F}\left(U,Q\right)\cdot\nabla\ell_{j}\left(x\right)\mathrm{d}x=-\intop_{\partial\varOmega_{e}}\mathbf{n}\cdot\mathbf{F}\left(U,Q\right)\ell_{j}\left(x\right)\mathrm{d}S`

and 

:math:`\intop_{\varOmega_{e}}Q^{e}\ell_{j}\left(x\right)\mathrm{d}x=\intop_{\Omega_{e}}\ell_{j}\left(x\right)\nabla Q^{e}\mathrm{d}x+\intop_{\partial\Omega}\mathbf{n}\left[V^{*}-V^{e}\right]\ell_{j}\left(x\right)\mathrm{d}S`

Since the solution is discontinuous across the interfaces, the fluxes are not defined at these 
interfaces. This requires the definition of a numerical flux to be defined at the interfaces. 
Different approaches exist for the definition of the fluxes which are solutions to the Riemann
problem. Currently two approaches are implemented, namely Lax-Friedrich and Roe solvers. Both 
of these flux options are upwinded which renders the spatial discreitzation numerically dissipative.
In practice this means that wave lengths that are not well resolved are dissipated which 
renders the scheme numerically stable (in a linear sense).

Once the domain is discretized the resulting system of equations is of the form:

:math:`\frac{\partial U}{\partial t}=R(U)`

which can be integrated with a time integrator
scheme. Different **explicit** temporal schemes are currently available in
``TPS``.

----

Capabilities
^^^^^^^^^^^^

A summary of the main flow-solver capabilities are:

Upwinding
  Roe and Lax-Friedrich interface luxes
  
HDF5 Output
  The solver outputs the solution at constant iteration intervals. Paraview and
  HDF5 ouputs are supported (the HDF5 variants are used when restarting the simulation).
  
Boundary Conditions
  Several boundary conditions are provided including:.
  
  * Adiabatic wall
  * Isothermal wall
  * Reflecting pressure outlet
  * Non-reflecting pressure outlet
  * Reflecting density and velocity input
  * Non-reflecting density and velocity input
  
Communication & computation overlap
  In parallel simulations, communication of shared data between computational domains is 
  communicated concurrently with the computation of the interior of the domain.
  
Restart with arbitrary # of MPI tasks and order
  When restarting the simulation it is possible to define a different polynomial order of the 
  solution. It is also possible to restart with a different number of MPI tasks (this requires
  creating an intermediary serialized version of solution first).
  
GPU variants
  It is also possible to run on GPU. Not all the features are available in the GPU version. In
  particular it is not possible to run with variable time-step. The non-reflecting inlet is
  similarly not available. See more details on the :doc:`build` page
  for details on enabling GPU support.

----

EM Solver
*********

Governing Equations
^^^^^^^^^^^^^^^^^^^

We support simulation of low-frequency electromagnetics using the
quasi-magnetostatic approximation of Maxwell's equations.
Specifically, neglecting the displacement current and using a magnetic
vector potential based formulation, the Ampere-Maxwell equation
becomes

:math:`\nabla \times \left( \mu^{-1} \nabla \times A \right) = J`,

where :math:`A` is the magnetic vector potential, :math:`\mu` is the
magnetic permeability, and :math:`J` is the current.  We assume that
the only conductors in the domain are those carrying the driving
current.  Thus, :math:`J` represents a user-specified source
current.  Further, the existing implementation is highly specialized
to approximate the electromagnetic environment in a plasma torch
geometry.  As such, we assume that 1) the magnetic permeability of all
materials in the geometry is same and 2) the source current is
composed of "rings" of uniform current density.  In this situation, it
is convenient to non-dimensionalize the equation using a reference
length :math:`\ell`, such as the ring radius, and the source current
magnitude :math:`J_0`.  The resulting non-dimensional governing
equation is given by

:math:`\hat{\nabla} \times \hat{\nabla} \times \hat{A} = \hat{J}`,

where :math:`\hat{A} = A/(\mu J_0 \ell^2)`, :math:`\hat{\nabla} = \ell
\nabla`, and :math:`\hat{J} = J/J_0`.  This is the equation approximated by
the solver.

Finally, perfect electrical conductor (PEC) boundary conditions are
assumed at all domain boundaries:

:math:`A \times n = 0`,

where :math:`n` denotes the outward pointing unit normal.

----

Numerics
^^^^^^^^

The weak form corresponding to the quasi-magnetostatic model with PEC
BCs is given by

:math:`\int_{\Omega} (\nabla \times v) \cdot (\nabla \times \hat{A}) \, \mathrm{d}x = \int_{\Omega} v \cdot \hat{J} \, \mathrm{d}x \quad \forall v \in H_0(curl; \Omega),`

where :math:`\Omega` denotes the computational domain and
:math:`H_0(curl; \Omega)` is the usual :math:`H(curl)` space with
homogeneous Dirichlet boundary conditions:

:math:`H_0(curl; \Omega) = \left\{ v \in L^2(\Omega)^3 | \nabla \times v \in L^2(\Omega)^3, \, v \times n|_{\partial \Omega} = 0\right\}`,

with :math:`\partial \Omega` denoting the boundary of the domain.

This weak form is discretized using :math:`H(curl)`-conforming
"Nedelec" finite element as provided by MFEM, leading to a sparse
linear system.  This linear system is solved using the mimimum
residuals (MINRES) method with Auxiliary-space Maxwell Solver (AMS)
preconditioner from Hypre (through MFEM).


