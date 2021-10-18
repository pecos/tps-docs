Overview
========

Introduction
************

The ``TPS`` code solves the Navier-Stokes (NS) equations or the Maxwell equations for 
Electro-Magnetics (EM) using the Finite Element Method (FEM). As the project evolves the
different equations will be solved simultaneously (fully coupled) so that efficient,
high-fidelity simulations of an Inductively Coupled Plasma (ICP) Torch can be made.

At the moment, a Discontinuous Galerkin approach has been taken for the solution of the NS
equations whereas a Continuous Galerkin (CG) approach has been taken for the solution of the
EM equations. These are implemented using the `MFEM <https://mfem.org>`_ library.




Flow Solver
************

----

Governing Equations
^^^^^^^^^^^^^^^^^^^


We consider the NS equations expressed in conservative form:

:math:`\frac{\partial U}{\partial t}+\nabla\cdot\mathbf{F}\left(U,\nabla V\right)=0`, where 

:math:`U=\left[\rho,\rho u,\rho v,\rho w,\rho E\right]^{T}` is the vector of conservative variables,
:math:`V=\left[\rho,u,v,w,p\right]^{T}` is the vector of primitive variables with pressure
:math:`p=\left(\gamma-1\right)\rho\left(E-\frac{1}{2}\left(u^{2}+v^{2}+w^{2}\right)\right)` 
and total energy
:math:`E=c_{v}T+\frac{1}{2}\left(u^{2}+v^{2}+w^{2}\right)` ; and 

:math:`\mathbf{F}=\left[F_{c}^{x}-F_{v}^{x},F_{c}^{y}-F_{v}^{y},F_{c}^{z}-F_{v}^{z}\right]` where

:math:`F_{c}^{i}, F_{v}^{i}` are the convective and viscous fluxes, respectively, defined as:

:math:`\left[F_{c}^{x},F_{c}^{y},F_{c}^{z}\right]=\left[\begin{array}{ccc}
\rho u & \rho v & \rho w\\
\rho u^{2}+p & \rho uv & \rho uw\\
\rho uv & \rho v^{2}+p & \rho vw\\
\rho uw & \rho vw & \rho w^{2}+p\\
u\left(\rho E+p\right) & v\left(\rho E+p\right) & w\left(\rho E+p\right)
\end{array}\right]`

:math:`\left[F_{v}^{x},F_{v}^{y},F_{v}^{z}\right]=\left[\begin{array}{ccc}
0 & 0 & 0\\
\tau_{xx} & \tau_{xy} & \tau_{xz}\\
\tau_{yx} & \tau_{yy} & \tau_{yz}\\
\tau_{zx} & \tau_{zy} & \tau_{zz}\\
u\tau_{xx}+v\tau_{yx}+w\tau_{zx}-q_{x} & u\tau_{xy}+v\tau_{yy}+w\tau_{zy}-q_{y} & u\tau_{xz}+v\tau_{yz}+w\tau_{zz}-q_{z}
\end{array}\right]`

where the stress tensor
:math:`\tau_{ij}` is defined as 
:math:`\tau_{ij}=\mu\left(\frac{\partial u_{i}}{\partial x_{j}}+\frac{\partial u_{j}}{\partial x_{i}}\right)-\frac{2}{3}\mu\nabla\cdot\mathbf{v}\delta_{ij}` 
and the heat flux :math:`\mathbf{q}=-k\nabla T` with thermal coefficient 
:math:`k=\frac{\mu c_{p}}{Pr}` where :math:`Pr` is the Prandtl number, estimated as :math:`Pr=0.71` for air. The viscosity coefficient follows the Sutherland law 
:math:`\mu=\frac{1.458\cdot10^{-6}T^{3/2}}{T+110.4}`.

The resulting system of equations is expressed as:
 
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

:math:`\frac{\partial U}{\partial t}=A(U)` 

which can be integrated with a time integrator
scheme. Different **explicit** temporal schemes are currently available in
``TPS``.

*Note that this is a research project and these numerical schemes may change in
future versons of the software.*

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
  
Restart with arbitrary # of MPU tasks and order
  When restarting the simulation it is possible to define a different polynomial order of the 
  solution. It is also possible to restart with a different number of MPI tasks (this requires
  creating an intermediary serialized version of solution first).
  
GPU variants
  It is also possible to run on GPU. Not all the features are available in the GPU version. In
  particular it is not possible to run with variable time-step. The non-reflecting inlet is
  similarly not available. See more details on the :ref:`build_page` page
  for details on enabling GPU support.

