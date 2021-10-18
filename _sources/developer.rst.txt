Developer Resources
===================

This page houses additional information targeting developers and maintainers of
the ``tps`` code base.

Coding Style
************

Two make targets exist to help promote some basic coding standards within the
code base. The coding style targets `Google's C++ Guide
<https://google.github.io/styleguide/cppguide.html>`_ and a static `linter <https://github.com/cpplint/cpplint>`_
targeting this style is configured to run with ``make style``::

  $ make style

  -------------------------------------------
  Checking source code style using cpplint.py
  -------------------------------------------

  Done processing BCintegrator.cpp
  Done processing BCintegrator.hpp
  Done processing BoundaryCondition.cpp
  Done processing BoundaryCondition.hpp
  Done processing M2ulPhyS.cpp
  Done processing M2ulPhyS.hpp
  Done processing averaging_and_rms.cpp
  Done processing averaging_and_rms.hpp
  Done processing dataStructures.hpp
  Done processing dgNonlinearForm.cpp
  Done processing dgNonlinearForm.hpp
  Done processing dgNonlinearForm.hpp
  Done processing domain_integrator.cpp
  Done processing domain_integrator.hpp
  Done processing equation_of_state.cpp
  Done processing equation_of_state.hpp
  Done processing faceGradientIntegration.cpp
  Done processing faceGradientIntegration.hpp
  Done processing face_integrator.cpp
  Done processing face_integrator.hpp
  Done processing fluxes.cpp
  Done processing fluxes.hpp
  Done processing forcing_terms.cpp
  Done processing forcing_terms.hpp
  Done processing gradNonLinearForm.cpp
  Done processing gradNonLinearForm.hpp
  Done processing gradients.cpp
  Done processing inletBC.cpp
  Done processing inletBC.hpp
  Done processing io.cpp
  Done processing io.hpp
  Done processing main.cpp
  Done processing masa_handler.cpp
  Done processing masa_handler.hpp
  Done processing mpi_groups.cpp
  Done processing mpi_groups.hpp
  Done processing outletBC.cpp
  Done processing outletBC.hpp
  Done processing rhs_operator.cpp
  Done processing rhs_operator.hpp
  Done processing riemann_solver.cpp
  Done processing riemann_solver.hpp
  Done processing run_configuration.cpp
  Done processing run_configuration.hpp
  Done processing sbp_integrators.cpp
  Done processing sbp_integrators.hpp
  Done processing tps.cpp
  Done processing tps.hpp
  Done processing utils.cpp
  Done processing wallBC.cpp
  Done processing wallBC.hpp

Note that this ``style`` check is executed on pull requests and must be
successful in order to land changes on the main branch.  Developers are
encouraged to run locally and everything required to perform this check is
included directly within the git repository.  If issues are detected, details
on the style exception will be displayed and you can fix by hand, or
alternatively, use an additional ``enforcestyle`` target to automatically fix
the majority of issues.  The ``enforcestyle`` check uses the `clang-format
<https://clang.llvm.org/docs/ClangFormat.html>`_ tool which must be installed
locally prior to running (this tool is part of a standard *clang* installation
and should be available within any standard Linux distro)::

  $ make enforcestyle

  ------------------------------------------------------
  Applying source code style updates using clang-format
  ------------------------------------------------------

  clang-format -i averaging_and_rms.hpp equation_of_state.hpp forcing_terms.hpp
  mpi_groups.hpp run_configuration.hpp BCintegrator.hpp
  faceGradientIntegration.hpp inletBC.hpp outletBC.hpp sbp_integrators.hpp
  BoundaryCondition.hpp face_integrator.hpp M2ulPhyS.hpp rhs_operator.hpp
  wallBC.hpp domain_integrator.hpp fluxes.hpp masa_handler.hpp
  riemann_solver.hpp dgNonlinearForm.hpp gradNonLinearForm.hpp
  dataStructures.hpp io.hpp tps.hpp averaging_and_rms.cpp
  faceGradientIntegration.cpp M2ulPhyS.cpp rhs_operator.cpp wallBC.cpp
  BCintegrator.cpp face_integrator.cpp riemann_solver.cpp BoundaryCondition.cpp
  fluxes.cpp masa_handler.cpp run_configuration.cpp domain_integrator.cpp
  forcing_terms.cpp mpi_groups.cpp sbp_integrators.cpp equation_of_state.cpp
  inletBC.cpp outletBC.cpp utils.cpp io.cpp dgNonlinearForm.cpp
  dgNonlinearForm.hpp gradients.cpp gradNonLinearForm.cpp tps.cpp main.cpp

Known Gotcha: The MFEM provided GPU macros (e.g. ``MFEM_FORALL_2D`` and
``MFEM_FOREACH_THREAD``) can cause
``clang-format`` to introduce odd indents in loops using these
macros. Consequently, consider disabling the formatter in regions using these
macros by embedding comments as follows:

.. code-block:: C++

   // clang-format off
   MFEM_FORALL_2D(n, dof, num_equation, 1, 1, {
     MFEM_FOREACH_THREAD(eq, x, num_equation) {
       ...

     }  // end MFEM_FOREACH_THREAD
   });  // end MFEM_FORALL_2D
   // clang-format on


Documentation
*************

This documentation site is built using `Sphinx <https://www.sphinx-doc.org>`_
with collateral hosted on the companion tps-docs `repository
<https://github.com/pecos/tps-docs>`_.  See the `README.md
<https://github.com/pecos/tps-docs/blob/main/README.md>`_ file for more
information on how to build the site locally and make changes to the production site.
