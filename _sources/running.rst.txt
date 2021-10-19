Running
========

The two primary inputs required to initiate the ``tps`` binary from scratch are:

#. mesh file
#. input options

Mesh Files
**********

The mesh file format should be supported by MFEM; we typically use `Gmsh's
<https://gmsh.info>`_ native "MSH" saved in an older 2.x format required by
MFEM.  Note that the older MSH format can be requested in Gmsh .geo files by including
the following stanza::

  Mesh.MshFileVersion = 2;

Example 2D and 3D meshes we use with regression testing are included in the
`test/meshes <https://github.com/pecos/tps/tree/main/test/meshes>`_ area of
the repository.

Boundary attributes
-------------------
In order to support boundary condition enforcement in the flow solver, the resulting meshes for
use with ``tps`` should include delineation of one or more boundary surfaces. This is
typically accomplished with Gmsh in 3D by defining physical surfaces for each
boundary you would like to treat separately in the corresponding .geo file. For
example, the following identifies two boundaries that map to Gmsh surfaces::

  Physical Surface (1) = {13}; 
  Physical Surface (2) = {12}; 

The type of boundary is not indicated in the MSH file, instead, the assigned
boundary numbers (e.g 1 and 2 in the above example) can be referenced by the
``tps`` runfile and assigned a desired type at runtime.

Volume attributes
-----------------
To enable specification of the source current, the EM implementation
assumes that the mesh has 5 volume attributes: 1 defining the
free-space (source current free) domain, and 4 defining source current
rings.  See the `QuasiMagnetostaticSolver <https://github.com/pecos/tps/tree/main/src/quasimagnetostatic.cpp>`_
class for details of the assumed source current.


Input options
*************

The flow and EM solvers handle input options separately.

Flow
----
For the flow solver, the *runfile* controls all aspects of the runtime configuration including
choice of input mesh, CFL and time integration settings, numerics settings, initial conditions,
and boundary condition definitions. At present,
this is a simple text-based inputfile (it will be refactored using a more
flexible *ini* style in future releases).

A specific input file must be specified on the command-line as follows::

  $ ./tps --runFile inputs/input.2iters.cyl

A variety of runfile examples are included in the `test/inputs/
<https://github.com/pecos/tps/tree/main/test/inputs>`_ subdirectory along with
a reference input file including more comments at `src/runfileExample.run
<https://github.com/pecos/tps/blob/main/src/runfileExample.run>`_.

Note that the ``patchNum`` input for available boundary conditions (e.g. Inlet,
Outlet, and Wall) should match the desired physical boundary surface assigned
in the mesh file.

Electromagnetics
----------------

For the EM solver, all input options are specified through the command
line.  To invoke the EM solver, use the ``--em-only`` option.
Additional command line options are as follows::

  Usage: ./src/tps [options] ...
  Options:
     -h, --help
  	Print this help message and exit.
     -flow, --flow-only, -nflow, --not-flow-only, current option: --flow-only
	Perform flow only simulation
     -run <string>, --runFile <string>, current value: <unknown>
	Name of the input file with run options.
     -v, --version, , --no-version, current option: --no-version
	Print code version and exit
     -em, --em-only, -nem, --not-em-only, current option: --not-em-only
	Perform electromagnetics only simulation
     -m <string>, --mesh <string>, current value: hello.msh
	Mesh file (for EM-only simulation)
     -o <int>, --order <int>, current value: 1
	Finite element order (polynomial degree) (for EM-only).
     -r <int>, --ref <int>, current value: 0
	Number of uniform refinements (for EM-only).
     -i <int>, --maxiter <int>, current value: 100
	Maximum number of iterations (for EM-only).
     -t <double>, --rtol <double>, current value: 1e-06
	Solver relative tolerance (for EM-only).
     -a <double>, --atol <double>, current value: 1e-10
	Solver absolute tolerance (for EM-only).
     -top, --top-only, -ntop, --no-top-only, current option: --no-top-only
	Run current through top branch only
     -bot, --bot-only, -nbot, --no-bot-only, current option: --no-bot-only
	Run current through bottom branch only
     -by <string>, --byfile <string>, current value: By.h5
	File for By interpolant output.
     -ny <int>, --nyinterp <int>, current value: 0
	Number of interpolation points.
     -y0 <double>, --yinterpMin <double>, current value: 0
	Minimum y interpolation value
     -y1 <double>, --yinterpMax <double>, current value: 1
	Maximum y interpolation value


Restart
*******

``tps`` will dump HDF5-based restart files at an iteration frequency dictated
by the runtime control <ITERS_OUT>.  These files can be used to continue an
existing simulation in time when restart mode is enabled (<RESTART_CYCLE>). By
default, the restart files are saved on a per MPI-process basis. In addition, a
companion domain-decomposition partitioning file will be generate to cache the
mesh partition configuration for a given MPI task count. The resulting file for
a 4-MPI task case looks as follows::

  partition.4p.h5

In normal restart mode, the continuation must be performed using the same
processor count that generated the restart files and the partition.*.h5 file
must be present in the same directory.  Note that ``tps`` will override the
solution restart files on each output, so care should be taken to preserve
previous solution files in an alternate directory if you desire to maintain
restart capability at a particular iteration count.

In addition to physical state variables, the HDF5 restart files also include metadata
to document the solution process including items like the total degrees of freedom,
timestep value, current iteration and time, and polynomial order.  These items
can be interrogated directly by name using standard HDF5 utilities; for example
to access the iteration count attribute, issue::

  $ h5dump -a iteration restart_output.sol.1.h5

  HDF5 "restart_output.sol.1.h5" {
  ATTRIBUTE "iteration" {
     DATATYPE  H5T_STD_I32LE
     DATASPACE  SCALAR
     DATA {
     (0): 10
     }
  }
  }

Alternate Processor Counts (N->M)
*********************************

In order to change processor counts during a long-running simulation, ``tps``
can be directed to generate a serialized restart solution that is housed in a
single file. To enable this, specify the <RESTART_SERIAL write> option in the
runfile. A typical use case when altering the task count is to just perform a
single iteration starting from *N* restart files, so set the <NMAX> setting
accordingly based on the starting restart files. Once complete, you can update
the runfile again to use the <RESTART_SERIAL read> mode and execute with *M*
MPI tasks (be sure to increase the <NMAX> setting).  This will read the serialized
restart and decompose the domain and solution onto *M* tasks. New restart files
will subsequently be saved across *M* files. The final step in the process is
to disable the <RESTART_SERIAL> option for the next job execution so that the
solution will continue to restart using the decomposed solution files from *M*
tasks.
  
Terminating Early
*****************

If a ``tps`` simulation is actively running, it will generally not complete
until the maximum time or iteration count values are reached (as specified in
the runfile).  If you would like to terminate the simulation immediately, you can create a
``DIE`` file in the running directory. This will trigger one final restart file
generation so that no timesteps will be lost and the code will subsequently
exit::

  $ touch DIE
  ...
  Detected DIE file, terminating early...
  HDF5 restart files mode: write

  Creating HDF5 group for defined IO families
  --> /solution : Solution state variables
    --> Saving (density)
    --> Saving (rho-u)
    --> Saving (rho-v)
    --> Saving (rho-w)
    --> Saving (rho-E)

Notes
*****

Additional notes and caveats running the solver are highlighted as follows:

#. GPU execution configurations must use a fixed time-step (set DT_CONSTANT)
#. The restart, processor count change, and early termination capabilities are currently only supported in the flow solver.
