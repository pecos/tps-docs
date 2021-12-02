Build
========

The ``TPS`` code bases utilizes an *autotools*-based build system. Before
beginning a build, please ensure that all the necessary software dependencies
are available locally. In general, you will need:

* `MFEM <https://github.com/mfem/mfem>`_ (and associated dependencies)
  
  * MPI C++ compiler 
  * `Hypre <https://github.com/hypre-space/hypre>`_ (v2.16.0 - v2.20.0)
  * `Metis <http://glaros.dtc.umn.edu/gkhome/metis/metis/download>`_
    
* `GRVY <https://github.com/hpcsi/grvy>`_
* `HDF5 <https://www.hdfgroup.org/downloads/hdf5/source-code/>`_

Additional details on building MFEM are available upstream at
https://mfem.org/building/. Note that for CPU-based execution, we often
leverage pre-built binaries for MFEM from the `OpenHPC
<https://mfem.org/building/>`_ project. Alternatively, for GPU-based execution, MFEM will need
to built locally with a backend that matches the desired GPU vendor
environment. To date, we have successfully used the **CUDA** and **HIP** back
ends.

----

Obtaining Source Code
*********************

To begin, start with a fresh clone of the ``TPS`` repository available at
https://github.com/pecos/tps (or from a release tarball).  Note that we store
larger binary files used within this repository using git `Large File Storage
(LFS) <https://git-lfs.github.com>`_ . Therefore, ``git-lfs`` is required to
resolve all files in the repository and most Linux distros provide this add-on
functionality. For OSX, you can install this binary via MacPorts or Brew.  You
can confirm a successful git clone configuration by running ``git lfs track``
which should return the list of files in the repository currently being tracked
by lfs.

----

**Initialization (git checkout only)** 


When starting directly from within a git checkout, you will need to first
bootstrap the autotools build system by running::

 $ ./bootstrap

Note that the above step is **not** necessary when starting from a release
tarball.

----

Configuration
*************

Next, use the ``configure`` script to define various compilation settings and
dependency library paths.  See ``configure --help`` for more information on
available options.  At a minimum, you will want to set the variables
**MFEM_INC** and **MFEM_LIB** to the paths of the desired MFEM installation to
resolve header and library files, respectively
Note that this variable may already be set when using pre-installed
versions of MFEM that ship with user-environment modules.  The ``configure``
step will differ depending on whether this is a CPU or GPU build. In the GPU
case, additional options are required to specify the desired architecture
types.  Several example ``configure`` examples are highlighted below:

**CPU example using OpenHPC provided MFEM**::

  $ module list
  Currently Loaded Modules:
    1) autotools          7) openblas/0.3.7  13) scalapack/2.1.0
    2) prun/2.0           8) superlu/5.2.1   14) petsc/3.14.4
    3) gnu9/9.3.0         9) hypre/2.18.1    15) ptscotch/6.0.6
    4) ohpc              10) metis/5.1.0     16) superlu_dist/6.1.1
    5) libfabric/1.10.1  11) phdf5/1.10.6    17) mfem/4.2
    6) mpich/3.3.2-ofi   12) netcdf/4.7.3
  
  $ ./configure -I$PETSC_INC LDFLAGS="-L$PETSC_LIB -lpetsc"

**GPU example using locally built MFEM with CUDA backend**

The ``CUDA_ARCH`` setting is required to define the target GPU and should match
the target execution hardware::

  $ export MFEM_INC=<yourpath/mfem-cuda>/include
  $ export MFEM_LIB=<yourpath>mfem-cuda/lib
  $ ./configure --enable-gpu-cuda CUDA_ARCH=sm_75

**GPU example using locally built MFEM with HIP backend**

Similarly, the ``HIP_ARCH`` setting is required to define the target GPU architecture::
  
  $ export MFEM_INC=<yourpath/mfem-hip>/include
  $ export MFEM_LIB=<yourpath>mfem-hip/lib
  $ ./configure --enable-gpu-hip HIP_ARCH=gfx803


**VPATH builds**

The build system also supports VPATH builds which allows for multiple build
configurations against the same source tree. To utilize this capability, create
a ``build`` directory and descend into this location prior to running
configure::

  $ mkdir build
  $ cd build
  $ ../configure <with desired options>
  

Compile
*******

After completing configuration, you can kick off the build using ``make``,
e.g.::

  $ make          # serial-build, or
  $ make -j 8     # parallel-build

Assuming the build completed successfully, the resulting solver binary should
be available in the ``src/`` subdirectory::

  $ ./src/tps --version

  ------------------------------------
    _______ _____   _____
   |__   __|  __ \ / ____|
      | |  | |__) | (___  
      | |  |  ___/ \___ \ 
      | |  | |     ____) | 
      |_|  |_|    |_____/ 
  
  TPS Version:  1.0 (dev)
  Git Version:  31e48b1
  MFEM Version: MFEM v4.2 (release)
  ------------------------------------
