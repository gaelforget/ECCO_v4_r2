
.. _runs:

Running ECCO Version 4
**********************

This section explains how the ECCO version 4 setup is used to re-run the
release 2 state estimate over 1992–2011 (:numref:`eccov4-baseline`), 
other solutions (:numref:`eccov4-other`), short regression tests (
:numref:`testreport`), or optimization tests (:numref:`optim`). 
Running MITgcm typically requires a linux cluster
with the following software: gcc, gfortran (or alternatives), mpi (for
parallel computation) and netcdf (for ‘pkg/profiles’). The `MITgcm
howto <http://mitgcm.org/public/devel_HOWTO/devel_HOWTO.pdf>`__ and
`MITgcm manual <http://mitgcm.org/public/r2_manual/latest/online_documents/manual.pdf>`__
provide additional information.

For users who may lack on-premise resources or IT support, an automated
recipe which leverages Amazon Web Services’ cfncluster technology and
sets up a complete computational environment in the cloud (hardware,
software, model, and inputs) is provided in the
`example_scripts/ <https://github.com/gaelforget/ECCO_v4_r2/tree/master/example_scripts/>`__
directory (under `ECCO v4
r2 <https://github.com/gaelforget/ECCO_v4_r2/>`__ in github). In a
January 2017 test, it ran the 20 year solution on 96 vCPUs within 36h
for a cost of about 40$ using AWS’ spot instances.

.. _mitgcmdirs:

.. rubric:: Organized Directories

includes the MITgcm as well as the ECCO v4 model setup and inputs, once they have been downloaded in ``MITgcm/mysetups`` according to the :numref:`download-setup` directions, so that they can be used according to the :numref:`eccov4-baseline` and the :ref:`baseline` directions. 

::

   MITgcm/
     model/     (core of MITgcm)
     pkg/       (MITgcm modules)
     tools/
       genmake2          (shell script)
       build_options     (wrt compilers)
     mysetups/           (user created)
       ECCO_v4_r2/
         build/
         code/
         input/
         input_itXX/
         results_itXX/
         forcing_baseline2/    (from wget)
         inputs_baseline2/     (from wget)

.. _eccov4-baseline:

The Release 2 Solution
----------------------

It is here assumed that MITgcm and ECCO v4 directories have been
downloaded and organized as shown in
:ref:`mitgcmdirs`. Users can then re-run the ECCO
version 4 release 2 solution by following the directions in
:ref:`baseline`. Afterwards they are strongly
encouraged to verify their results by using the included
testreport_ecco.m Matlab script as depicted in
:ref:`testreportecco`. The expected level of
accuracy for 20-year re-runs, based upon an up-to-date MITgcm code and a
standard computing environment, is reached when the displayed values are
all :math:`\leq-3`. Interpretation of the testreport_ecco.m output is
explained in detail in :cite:`for-eta:15`.

The 20-year model run typically takes between 6 to 12 hours of
wall-clock time on 96 cores using a modern computing environment. The
number of cores is 96 by default as reflected by
:ref:`baseline` but can be reduced to 24 simply by
copying ‘ECCO_v4_r2/code/SIZE.h_24cores’ over ‘ECCO_v4_r2/code/SIZE.h’
before compiling the model and then running it with ‘-np 24’ rather than
‘-np 96’ in :ref:`baseline`. However, it should be
noted that reducing the number of cores increases wall-clock time and
memory requirements.

.. _baseline:

.. rubric:: Compile, Link, Run

Procedure to compile MITgcm and re-run the ECCO v4 r2 solution :cite:`dspace-eccov4r2`. Pre-requisites: (1) gcc, gfortran (or alternatives), mpi (for parallel computation) and netcdf (for pkg/profiles); (2) MITgcm and ECCO v4 setup (:numref:`download-setup`); (3) input directories organized as shown in :ref:`mitgcmdirs` (see :numref:`download-setup`). Other compiler options, besides linux_amd64_gfortran, are provided by the MITgcm development team in MITgcm/tools/build_options/ for cases when gfortran is not available. The contents of inputs_baseline2/ should match this `ftp server <ftp://mit.ecco-group.org/ecco_for_las/version_4/release2/input_ecco/>`__ (see :numref:`download-setup`).

::

    #1) compile model
    cd MITgcm/mysetups/ECCO_v4_r2/build
    ../../../tools/genmake2 -mods=../code -optfile \
         ../../../tools/build_options/linux_amd64_gfortran -mpi
    make depend
    make -j 4
    cd ..

    #2) link files into run directory
    mkdir run
    cd run
    ln -s ../build/mitgcmuv .
    ln -s ../input/* .
    ln -s ../inputs_baseline2/input*/* .
    ln -s ../forcing_baseline2 .

    #3) run model
    mpiexec -np 96 ./mitgcmuv

.. _testreportecco:

.. rubric:: Verify Run Accuracy

Top: instructions to gauge the accuracy of a re-run of ECCO v4 r2 :cite:`dspace-eccov4r2` using the testreport_ecco.m Matlab script :cite:`for-eta:15`. Bottom: sample output of testreport_ecco.m where the re-run agrees up to 6 digits with the reference result. Additional tests of meridional transports can be activated by users who have installed the gcmfaces toolbox :cite:`for-eta:15` as explained in :numref:`download-analysis`. To this end, users would uncomment the ``addpath ~/Documents/MATLAB/gcmfaces;`` and ``gcmfaces_global;`` commands below and, if needed, replace ``~/Documents/MATLAB/gcmfaces`` with the location where gcmfaces has been installed.

::


    cd MITgcm/mysetups/ECCO_v4_r2
    matlab -nodesktop -nodisplay

    %addpath ~/Documents/MATLAB/gcmfaces;
    %gcmfaces_global;

    addpath results_itXX;%add necessary .m and .mat files to path
    mytest=testreport_ecco('run/');%compute tests and display results

::

    --------------------------------------------------------------
           &   jT &   jS &      ... &  (reference is)
    run/   & (-6) & (-6) &      ...  &  baseline2      
    --------------------------------------------------------------

.. _eccov4-other:

Other 20-Year Solutions
-----------------------

It is here assumed that MITgcm and ECCO v4 directories have been
downloaded and organized as shown in
:ref:`mitgcmdirs`. Users can then re-run the
‘baseline 1’ solution that more closely matches the original, release 1,
solution of :cite:`for-eta:15`. However, to re-run
baseline 1 instead of release 2, a few modifications to the setup are
needed: (a) download the corresponding forcing fields as follows:

::

    wget --recursive ftp://mit.ecco-group.org/ecco_for_las/version_4/release1/forcing_baseline1/

(b) before compiling the model: define ‘ALLOW_KAPGM_CONTROL_OLD’ and
‘ALLOW_KAPREDI_CONTROL_OLD’ in ‘ECCO_v4_r2/code/GMREDI_OPTIONS.h’;
define ‘ALLOW_AUTODIFF_INIT_OLD’ in
‘ECCO_v4_r2/code/AUTODIFF_OPTIONS.h’; (c) before running the model: copy
‘ECCO_v4_r2/input_itXX/data’ and ‘data.exf’ over ‘ECCO_v4_r2/input/data’
and ‘data.exf’.

Users who may want to reproduce ‘release1’ even more precisely than
‘baseline1’ does should contact ecco-support@mit.edu to obtain
additional model inputs. Users holding a
`TAF <http://www.fastopt.de/>`__ license can also: (a) compile the
adjoint by replacing ‘make -j 4’ with ‘make adall -j 4’ in
:ref:`baseline`; (b) activate the adjoint by setting
‘useAUTODIFF=.TRUE.,’ in data.pkg; (c) run the adjoint by replacing
‘mitgcmuv’ with ‘mitgcmuv_ad’ in :ref:`baseline`.

.. _testreport:

Short Forward Tests
-------------------

To ensure continued compatibility with the up to date MITgcm, the ECCO
v4 model setup is also tested on a daily basis using the MITgcm’s
testreport command line utility (indicated in
:ref:`mitgcmdirs`) that compares re-runs with
reference results over a few time steps (see below for guidance and `the
MITgcm howto <http://mitgcm.org/public/devel_HOWTO/devel_HOWTO.pdf>`__
for additional details). These tests use dedicated versions of the ECCO
v4 model setup which are located within
`MITgcm_contrib/verification_other/ <http://mitgcm.org/viewvc/MITgcm/MITgcm_contrib/verification_other/>`__.

`global_oce_llc90/ <http://mitgcm.org/viewvc/MITgcm/MITgcm_contrib/verification_other/global_oce_llc90/>`__
(595M) uses the same LLC90 grid as the production ECCO v4 setup does
(section `2.1 <#eccov4-baseline>`__). Users are advised against running
forward tests using fewer than 12 cores (96 for adjoint tests) to avoid
potential memory overloads.
`global_oce_cs32/ <http://mitgcm.org/viewvc/MITgcm/MITgcm_contrib/verification_other/global_oce_cs32/>`__
(614M) uses the much coarser resolution CS32 grid and can thus be used
on any modern laptop. Instructions for their installation are provided
in `this
README <http://mitgcm.org/viewvc/*checkout*/MITgcm/MITgcm_contrib/verification_other/global_oce_llc90/README>`__
and `that
README <http://mitgcm.org/viewvc/*checkout*/MITgcm/MITgcm_contrib/verification_other/global_oce_cs32/README>`__,
respectively. Once installed, the smaller setup for instance can be
executed on one core by typing:

::

    cd MITgcm/verification/
    ./testreport -t global_oce_cs32

If everything proceeds as expected then the results are reported to
screen as shown in :ref:`report`. The daily results of the
regression tests (ran on the ‘glacier’ cluster) are reported `on this
site <http://mitgcm.org/public/testing.html>`__. On other machines the
degree of agreement (16 digits in ref:`report`) may vary
and testreport may indicate ‘FAIL’. Note: despite the seemingly dramatic
character of this message, users may still be able to reproduce 20-year
solutions with acceptable accuracy (:numref:`eccov4-baseline`).
To test
`global_oce_llc90/ <http://mitgcm.org/viewvc/MITgcm/MITgcm_contrib/verification_other/global_oce_llc90/>`__
using 24 processors and gfortran the corresponding command typically is:

::

    cd MITgcm/verification/
    ./testreport -of ../tools/build_options/linux_amd64_gfortran \
    -j 4 -MPI 24 -command 'mpiexec -np TR_NPROC ./mitgcmuv' \
    -t global_oce_llc90

.. _report:

.. rubric:: Verify Short Test

Below is an abbreviated example of testreport output to screen.

::

    default 10  ----T-----  ----S-----  
    G D M    c        m  s        m  s  
    e p a R  g  m  m  e  .  m  m  e  . 
    n n k u  2  i  a  a  d  i  a  a  d  
    2 d e n  d  n  x  n  .  n  x  n  . 

    Y Y Y Y>14<16 16 16 16 16 16 16 16  pass  global_oce_cs32

.. _optim:

Other Short Tests
-----------------

Running the adjoint tests associated with
:numref:`testreport` requires: (1) a
`TAF <http://www.fastopt.de/>`__ license; (2) to soft link ‘code’ as
‘code_ad’ in
`global_oce_cs32/ <http://mitgcm.org/viewvc/MITgcm/MITgcm_contrib/verification_other/global_oce_cs32/>`__
and
`global_oce_llc90/ <http://mitgcm.org/viewvc/MITgcm/MITgcm_contrib/verification_other/global_oce_llc90/>`__.
Users that hold a TAF license can then further proceed with the
iterative optimization test case in
`global_oce_cs32/input_OI/ <http://mitgcm.org/viewvc/MITgcm/MITgcm_contrib/verification_other/global_oce_cs32/input_OI>`__.
Here the ocean model is replaced with a simple diffusion equation.

The pre-requisites are:

#. run the adjoint benchmark in
   `global_oce_cs32/ <http://mitgcm.org/viewvc/MITgcm/MITgcm_contrib/verification_other/global_oce_cs32/>`__
   via testreport (see section `2.3 <#testreport>`__).

#. Go to MITgcm/lsopt/ and compile (see section 3.18 of
   `manual <http://mitgcm.org/public/r2_manual/latest/online_documents/manual.pdf>`__).

#. Go to MITgcm/optim/, replace ‘natl_box_adjoint’ with
   ‘global_oce_cs32’ in `this
   Makefile <http://mitgcm.org/viewvc/MITgcm/optim/Makefile>`__, and
   compile as explained in section 3.18 of
   `manual <http://mitgcm.org/public/r2_manual/latest/online_documents/manual.pdf>`__.
   An executable named ‘optim.x’ should get created in MITgcm/optim. If
   otherwise, please contact mitgcm-support@mit.edu

#. go to MITgcm/verification/global_oce_cs32/input_OI/ and type ‘source
   ./prepare_run’

To match the reference results reported in `this
file <http://mitgcm.org/viewvc/*checkout*/MITgcm/MITgcm_contrib/verification_other/global_oce_cs32/input_OI/README>`__,
users should proceed as follows

#. ./mitgcmuv_ad :math:`>` output.txt

#. ./optim.x :math:`>` op.txt

#. increment optimcycle by 1 in data.optim

#. go back to step #1 to run the next iteration

#. type ‘grep fc costfunction000\*’ to display results


