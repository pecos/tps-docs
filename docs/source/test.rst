Testing
========

The TPS code base maintains a suite of regression tests to help support
ongoing development. Once the ``tps`` binary is successfully built, you can
exercise the test suite locally to confirm the code is working
satisfactorily. The test-suite relies on the `Bats
<https://github.com/bats-core/bats-core>`_ unit testing framework and a copy is
included within the TPS repository. 
Individual tests are housed under the ``test/`` subdirectory (demarcated via
the *.test* suffix).  To run the entire suite, issue:

.. code-block:: text

  $ make check
  ...
  PASS: vpath.sh
  PASS: cyl3d.test 1 [cyl3d] check for input file inputs/input.4iters.cyl
  PASS: cyl3d.test 2 [cyl3d] run tps with input -> inputs/input.4iters.cyl
  PASS: cyl3d.test 3 [cyl3d] verify tps output with input -> inputs/input.4iters.cyl
  PASS: cyl3d.test 4 [cyl3d] verify git sha in HDF5 file
  PASS: cyl3d.test 5 [cyl3d] verify exit code if restart files missing
  PASS: cyl3d.test 6 [cyl3d] verify consistent solution with restart from 2 iters
  PASS: cyl3d.test 7 [cyl3d] verify tps output after variable p restart
  PASS: cyl3d.test 8 [cyl3d] verify serial restart consistency using 2 mpi tasks
  PASS: cyl3d.test 9 [cyl3d] verify serial restart consistency using 4 mpi tasks
  ...
  PASS: wedge.test 1 [2d/wedge] check for input file inputs/input.2d.wedge
  PASS: wedge.test 2 [2d/wedge] run tps with input -> inputs/input.2d.wedge
  PASS: wedge.test 3 [2d/wedge] verify computed inlet area in output -> wedge.log
  PASS: wedge.test 4 [2d/wedge] verify # of inlet faces detected -> wedge.log
  PASS: wedge.test 5 [2d/wedge] verify computed inlet outlet in output -> wedge.log
  PASS: wedge.test 6 [2d/wedge] verify # of outlet faces detected -> wedge.log
  ============================================================================
  Testsuite summary for tps 1.0 (dev)
  ============================================================================
  # TOTAL: 32
  # PASS:  32
  # SKIP:  0
  # XFAIL: 0
  # FAIL:  0
  # XPASS: 0
  # ERROR: 0
  ============================================================================

Note that individual tests can be run by simply executing the associated Bats
script. This can be helpful to exercise when a test fails and you want to see
which specific portions are failing:

.. code-block:: text

  $ ./cyl3d.mflow.test
  ✓ [cyl3d.mflow.outlet] check for input file inputs/input.2iters.mflow.cyl
  ✓ [cyl3d.mflow.outlet] verify outlet area calculation with input -> inputs/input.2iters.mflow.cyl
  ✓ [cyl3d.mflow.outlet] verify tps output with input -> inputs/input.2iters.mflow.cyl
  ✓ [cyl3d.mflow.outlet] verify tps output with input -> inputs/input.2iters.mflow.bulkVisc.dtconst.cyl

Continuous Integration (CI)
***************************

Thee regression tests highlighted above are exercised automatically as part of
the CI configuration enabled for pull requests using `GitHub Actions
<https://github.com/pecos/tps/actions/workflows/build.yaml>`_ . Underlying
tests differ slightly depending on whether ``TPS`` is configured for CPU or GPU
execution and the current CI configuration exercises CPU and two GPU variants
(Cuda/HIP) on each pull request.
