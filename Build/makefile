# Makefile for Fire Dynamics Simulator (FDS)

# To use this makefile, select the appropriate compiler and OS from the list below. For example, to
# compile the parallel version (MPI) of FDS under Linux using the Intel compilers type:
#
# make impi_intel_linux

# A make utility for windows may be downloaded from http://gnuwin32.sourceforge.net/packages/make.htm

# The variable VPATH defines where the source code is relative to the current directory

VPATH = ../Source

# 3rd part library versions
HYPRE_VERSION=v2.33.0
SUNDIALS_VERSION=v6.7.0
HDF5_VERSION=v1.14.5

ifeq ($(shell echo "check_quotes"),"check_quotes")
# windows
  GIT_HASH := $(shell ..\Scripts\githash)
  GIT_STAT := $(shell ..\Scripts\gitstat)
  GIT_DATE := $(shell ..\Scripts\gitlog)
  GIT_BRANCH := $(shell ..\Scripts\gitbranch)
  BUILD_DATE := $(shell powershell -Command "(Get-Date).ToString('ddd-MM/dd/yyyy-hh:mm-tt')")
else
# linux/osx
  SHELL:=/bin/bash
  GIT_HASH := $(shell git describe --long --abbrev=7)
  ifeq ("$(shell git diff --shortstat $(VPATH)/*.f90)","")
    GIT_STAT :=
  else
    GIT_STAT := -dirty
  endif
  GIT_DATE := $(shell git log -1 --format=%cd)
  GIT_BRANCH := $(shell git rev-parse --abbrev-ref HEAD)
  BUILD_DATE := $(shell date "+%b %d, %Y  %T")
# The preprocessor used by xlf uses comma as separation between preprocessing commands.
# Commas needs to be escaped if they are to be used as part of a value in a preprocessing variable.
  BUILD_DATE_XLF := $(shell date "+%b %d\, %Y  %T")
# For OSX get CLT version:
  CLT_VERSION := $(shell ../Scripts/osx_clt.sh)
endif

HYPRE_INFO=-DHYPRE_PP=\"$(HYPRE_VERSION)\"
SUNDIALS_INFO=-DSUNDIALS_PP=\"$(SUNDIALS_VERSION)\"
HDF5_INFO=-DHDF5_PP=\"$(HDF5_VERSION)\"

GITINFO_BASE=-DGITHASH_PP=\"$(GIT_HASH)$(GIT_STAT)-$(GIT_BRANCH)\" -DGITDATE_PP=\""$(GIT_DATE)\""
GITINFO=-fpp $(GITINFO_BASE) -DBUILDDATE_PP=\""$(BUILD_DATE)\""
GITINFOGNU=-cpp $(GITINFO_BASE) -DBUILDDATE_PP=\""$(BUILD_DATE)\""
GITINFOXLF=-qsuffix=cpp=f90 '-WF,-DGITHASH_PP=\"$(GIT_HASH)\",-DGITDATE_PP=\"$(GIT_DATE)\",-DBUILDDATE_PP=\"$(BUILD_DATE_XLF)\"'
GITINFOPGI=-Mpreprocess $(GITINFO_BASE) -DBUILDDATE_PP=\""$(BUILD_DATE)\""
GFORTRAN_OPTIONS=

FFLAGSMKL =
LFLAGSMKL_INTEL =
LFLAGSMKL_INTEL_OPENMP =
LFLAGSMKL_GNU_OPENMP =
LFLAGSMKL_OMPI_INTEL_LINUX =
LFLAGSMKL_CUSTOM =
LFLAGSMKL_CUSTOM_OPENMP =

FFLAGS_SUNDIALS =
LFLAGS_SUNDIALS =
FFLAGS_HYPRE =
LFLAGS_HYPRE =

ifdef MKLROOT # This assumes the MKL library is available.
#Set MKLLIBDIR
MKL_DIR_UNIX  := $(subst \,/,$(MKLROOT))
MKL_DIR_INTEL := $(MKL_DIR_UNIX)/lib/intel64
MKL_DIR_BASE  := $(MKL_DIR_UNIX)/lib
MKL_BLA_LIB   := mkl_blacs_intelmpi_lp64
MKL_BLA_PATH_A := "$(MKL_DIR_INTEL)/lib$(MKL_BLA_LIB).a"
MKL_BLA_PATH_LIB := "$(MKL_DIR_INTEL)/$(MKL_BLA_LIB).lib"
ifneq (,$(wildcard $(MKL_BLA_PATH_A) $(MKL_BLA_PATH_LIB)))
    MKLLIBDIR := $(MKL_DIR_INTEL)
else
    MKLLIBDIR := $(MKL_DIR_BASE)
endif
$(info MKLLIBDIR = $(MKLLIBDIR))

ifeq ($(shell echo "check_quotes"),"check_quotes")
  # windows
  FFLAGSMKL = -DWITH_MKL /I"$(MKLROOT)\include"
  COMMON_MKL_LIBS = "$(MKLLIBDIR)\mkl_intel_lp64.lib" "$(MKLLIBDIR)\mkl_core.lib" "$(MKLLIBDIR)\mkl_blacs_intelmpi_lp64.lib"
  LFLAGSMKL_INTEL = $(COMMON_MKL_LIBS) "$(MKLLIBDIR)\mkl_sequential.lib"
  LFLAGSMKL_INTEL_OPENMP = $(COMMON_MKL_LIBS) "$(MKLLIBDIR)\mkl_intel_thread.lib"
else
# linux
  FFLAGSMKL = -DWITH_MKL -I$(MKLROOT)/include
  COMMON_MKL_LIBS = ${MKLLIBDIR}/libmkl_intel_lp64.a ${MKLLIBDIR}/libmkl_core.a -lpthread -lm -ldl

  # impi_intel_linux targets
  COMMON_MKL_INTEL_LIBS = ${COMMON_MKL_LIBS} ${MKLLIBDIR}/libmkl_blacs_intelmpi_lp64.a
  LFLAGSMKL_INTEL = -Wl,--start-group ${COMMON_MKL_INTEL_LIBS} ${MKLLIBDIR}/libmkl_sequential.a -Wl,--end-group
  LFLAGSMKL_INTEL_OPENMP = -Wl,--start-group ${COMMON_MKL_INTEL_LIBS} ${MKLLIBDIR}/libmkl_intel_thread.a -liomp5 -Wl,--end-group
  
  # ompi_gnu_linux targets
  COMMON_MKL_GNU_LIBS = ${MKLLIBDIR}/libmkl_gf_lp64.a ${MKLLIBDIR}/libmkl_core.a ${MKLLIBDIR}/libmkl_blacs_openmpi_lp64.a -lpthread -lm -ldl
  LFLAGSMKL_GNU_OPENMP = -Wl,--start-group ${COMMON_MKL_GNU_LIBS} ${MKLLIBDIR}/libmkl_gnu_thread.a -lgomp -Wl,--end-group
 
  # ompi_intel_linux target
  LFLAGSMKL_OMPI_INTEL_LINUX = -Wl,--start-group ${COMMON_MKL_LIBS} ${MKLLIBDIR}/libmkl_sequential.a ${MKLLIBDIR}/libmkl_blacs_openmpi_lp64.a  -Wl,--end-group
  
  # ompi_gnu_osx and ompi_intel_osx targets
  COMMON_MKL_CUSTOM_LIBS = ${COMMON_MKL_LIBS} ${MKLLIBDIR}/libmkl_blacs_custom_lp64.a
  LFLAGSMKL_CUSTOM = ${COMMON_MKL_CUSTOM_LIBS} ${MKLLIBDIR}/libmkl_sequential.a
  LFLAGSMKL_CUSTOM_OPENMP = ${COMMON_MKL_CUSTOM_LIBS} ${MKLLIBDIR}/libmkl_intel_thread.a
 endif # OS test
endif # MKLROOT test


ifdef SUNDIALS_HOME # This assumes the SUNDIALS library is available.
   FFLAGS_SUNDIALS = -DWITH_SUNDIALS ${SUNDIALS_INFO} -I"${SUNDIALS_HOME}/fortran"
   FFLAGS_SUNDIALS_WIN = -DWITH_SUNDIALS ${SUNDIALS_INFO} /I"${SUNDIALS_HOME}\fortran"
   LFLAGS_SUNDIALS = ${SUNDIALS_HOME}/lib/libsundials_fcvode_mod.a ${SUNDIALS_HOME}/lib/libsundials_fnvecserial_mod.a ${SUNDIALS_HOME}/lib/libsundials_cvode.a
   LFLAGS_SUNDIALS_WIN = ${SUNDIALS_HOME}\lib\sundials_fcvode_mod.lib ${SUNDIALS_HOME}\lib\sundials_fnvecserial_mod.lib ${SUNDIALS_HOME}\lib\sundials_cvode.lib
endif

ifdef HYPRE_HOME # This assumes the HYPRE library is available.
   FFLAGS_HYPRE = -DWITH_HYPRE ${HYPRE_INFO} -I${HYPRE_HOME}/include
   FFLAGS_HYPRE_WIN = -DWITH_HYPRE ${HYPRE_INFO} /I"${HYPRE_HOME}\include"
   ifeq ($(BUILD_WITH_GPU), ON)
      FFLAGS_HYPRE += -DWITH_HYPRE_DEVICE
      FFLAGS_HYPRE_WIN += -DWITH_HYPRE_DEVICE
   endif
   LFLAGS_HYPRE = -L${HYPRE_HOME}/lib -lHYPRE -lm
   LFLAGS_HYPRE_WIN = ${HYPRE_HOME}\lib\HYPRE.lib
endif

FFLAGS_HDF5 =
LFLAGS_HDF5 =
ifdef HDF5_HOME # This assumes the HDF5 libary is available
   FFLAGS_HDF5 = -DWITH_HDF5 ${HDF5_INFO} -I"${HDF5_HOME}/include"
   FFLAGS_HDF5_WIN = -DWITH_HDF5 ${HDF5_INFO} /I"${HDF5_HOME}\include"
   LFLAGS_HDF5 = ${HDF5_HOME}/lib/libhdf5_fortran.a ${HDF5_HOME}/lib/libhdf5_f90cstub.a ${HDF5_HOME}/lib/libhdf5.a -lz
   LFLAGS_HDF5_WIN = ${HDF5_HOME}\lib\libhdf5_fortran.lib ${HDF5_HOME}\lib\libhdf5_f90cstub.lib ${HDF5_HOME}\lib\libhdf5.lib
endif # HDF5_HOME

LFLAGS_LINK_WIN =
linkflag = false
ifdef SUNDIALS_HOME
   linkflag = true
endif
ifdef HDF5_HOME
   linkflag = true
endif
ifeq ($(linkflag),true)
   LFLAGS_LINK_WIN = /link /NODEFAULTLIB:MSVCRTD /NODEFAULTLIB:libcmtd.lib Shlwapi.lib
endif


LFLAGS_GPU =
ifeq ($(BUILD_WITH_GPU), ON)
   ifeq ($(GPU_ARCH), hip)
      LFLAGS_GPU = -L/usr/lib64 -lstdc++ -L${ROCM_DIR}/lib -lamdhip64 -lrocsparse -lrocblas -lrocsolver -lrocrand
   else ifeq ($(GPU_ARCH), cuda)
      LFLAGS_GPU = -ldl -L/usr/lib64 -lstdc++ -L${CUDA_DIR}/lib64 -lcudart -lcusparse -lcublas -lcurand -L${CUDA_MATH_DIR}/lib64 -lcusolver
      LFLAGS_GPU_WIN = "$(GPU_HOME)\lib\x64\cudart.lib" "$(GPU_HOME)\lib\x64\cusparse.lib" "$(GPU_HOME)\lib\x64\cublas.lib" "$(GPU_HOME)\lib\x64\curand.lib" "$(GPU_HOME)\lib\x64\cusolver.lib"
   else ifeq ($(GPU_ARCH), sycl)
      LFLAGS_GPU = -L/usr/lib64 -lstdc++ -L${MKLROOT}/lib -lmkl_sycl -L${SYCL_DIR}/lib -lsycl
   endif
endif

M64_FLAG = -m64
ifeq ($(NO_M64_FLAG), ON)
   M64_FLAG =
endif

obj_mpi = prec.o cons.o chem.o prop.o devc.o type.o data.o mesh.o func.o gsmv.o smvv.o rcal.o turb.o soot.o \
          pois.o geom.o ccib.o radi.o part.o vege.o ctrl.o hvac.o mass.o imkl.o \
          wall.o fire.o velo.o pres.o init.o vtkf.o dump.o read.o divg.o main.o

objwin_mpi = $(obj_mpi:.o=.obj)

# General Purpose Rules

no_target:
	@echo \******** You did not specify a make target \********
	@echo Please read the comments at the top of the makefile

setup:
%.o : %.mod

setup_win:
%.obj : %.mod

.SUFFIXES: .f90 .o .obj

.f90.o:
	$(FCOMPL) -c $(FFLAGS) $<
.f90.obj:
	$(FCOMPL) -c $(FFLAGS) $<

# Files which have OpenMP compiler directives are compiled using the FOPENMPFLAG and can be enabled/disabled on a per-file basis.

ccib.o: FFLAGS += $(FOPENMPFLAGS)
divg.o: FFLAGS += $(FOPENMPFLAGS)
func.o: FFLAGS += $(FOPENMPFLAGS)
mass.o: FFLAGS += $(FOPENMPFLAGS)
pres.o: FFLAGS += $(FOPENMPFLAGS)
turb.o: FFLAGS += $(FOPENMPFLAGS)
velo.o: FFLAGS += $(FOPENMPFLAGS)
radi.o: FFLAGS += $(FOPENMPFLAGS)
wall.o: FFLAGS += $(FOPENMPFLAGS)
main.o: FFLAGS += $(FOPENMPFLAGS)

ccib.obj: FFLAGS += $(FOPENMPFLAGS)
divg.obj: FFLAGS += $(FOPENMPFLAGS)
func.obj: FFLAGS += $(FOPENMPFLAGS)
mass.obj: FFLAGS += $(FOPENMPFLAGS)
pres.obj: FFLAGS += $(FOPENMPFLAGS)
turb.obj: FFLAGS += $(FOPENMPFLAGS)
velo.obj: FFLAGS += $(FOPENMPFLAGS)
radi.obj: FFLAGS += $(FOPENMPFLAGS)
wall.obj: FFLAGS += $(FOPENMPFLAGS)
main.obj: FFLAGS += $(FOPENMPFLAGS)

# Add flags for VTKF compilation
#vtkf.o: FFLAGS += -D_R16P
#vtkf.obj: FFLAGS += -D_R16P
#vtkf.o: FFLAGS += -D_R_P_IS_R4P -D_ASCII_SUPPORTED


# Windows Intel Fortran Compiler and Intel MPI
#*** Note: the script, Build\Scripts\setup_intel_compilers.bat, must be run before running make with this target

impi_intel_win : MPILIB = "$(I_MPI_ROOT)\lib\impi.lib"
impi_intel_win : MPIINCLUDE = "$(I_MPI_ROOT)\include\mpi"
impi_intel_win : FFLAGS = -D_WIN32 /Qipo /O2 /I$(MPIINCLUDE) /wrap-margin- $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE_WIN) $(FFLAGS_SUNDIALS_WIN) $(FFLAGS_HDF5_WIN) /Qdiag-disable:10448
impi_intel_win : FCOMPL = $(COMP_FC)
impi_intel_win : obj = fds_impi_intel_win
impi_intel_win : setup_win $(objwin_mpi)
	$(FCOMPL) -o $(obj) $(FFLAGS) /F1000000000 $(objwin_mpi) $(MPILIB) $(LFLAGSMKL_INTEL) $(LFLAGS_HYPRE_WIN) $(LFLAGS_SUNDIALS_WIN) $(LFLAGS_HDF5_WIN) $(LFLAGS_LINK_WIN) $(LFLAGS_GPU_WIN)

impi_intel_win_vt : MPILIB = "$(I_MPI_ROOT)\lib\impi.lib"
impi_intel_win_vt : MPIINCLUDE = "$(I_MPI_ROOT)\include\mpi"
impi_intel_win_vt : FFLAGS = -D_WIN32 /Qipo /O2 /I$(MPIINCLUDE) /wrap-margin- $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE_WIN) $(FFLAGS_SUNDIALS_WIN) $(FFLAGS_HDF5_WIN) /Qdiag-disable:10448 /Zi /D "TBB_USE_THREADING_TOOLS"
impi_intel_win_vt : FCOMPL = $(COMP_FC)
impi_intel_win_vt : obj = fds_impi_intel_win_vt
impi_intel_win_vt : setup_win $(objwin_mpi)
	$(FCOMPL) -o $(obj) $(FFLAGS) /F1000000000 $(objwin_mpi) $(MPILIB) $(LFLAGSMKL_INTEL) $(LFLAGS_HYPRE_WIN) $(LFLAGS_SUNDIALS_WIN) $(LFLAGS_HDF5_WIN) $(LFLAGS_LINK_WIN)


impi_intel_win_openmp : MPILIB = "$(I_MPI_ROOT)\lib\impi.lib"
impi_intel_win_openmp : MPIINCLUDE = "$(I_MPI_ROOT)\include\mpi"
impi_intel_win_openmp : FFLAGS = -D_WIN32 /Qipo /O2 /I$(MPIINCLUDE) /wrap-margin- $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE_WIN) $(FFLAGS_SUNDIALS_WIN) $(FFLAGS_HDF5_WIN) /Qdiag-disable:10448
impi_intel_win_openmp : FOPENMPFLAGS = /Qopenmp
impi_intel_win_openmp : FCOMPL = $(COMP_FC)
impi_intel_win_openmp : obj = fds_impi_intel_win_openmp
impi_intel_win_openmp : setup_win $(objwin_mpi)
	$(FCOMPL) -o $(obj) $(FFLAGS) $(FOPENMPFLAGS) /F1000000000 $(objwin_mpi) $(MPILIB) $(LFLAGSMKL_INTEL_OPENMP) $(LFLAGS_HYPRE_WIN) $(LFLAGS_SUNDIALS_WIN) $(LFLAGS_HDF5_WIN) $(LFLAGS_LINK_WIN)

impi_intel_win_dv : MPILIB = "$(I_MPI_ROOT)\lib\impi.lib"
impi_intel_win_dv : MPIINCLUDE = "$(I_MPI_ROOT)\include\mpi"
impi_intel_win_dv : FFLAGS = -D_WIN32 /O1 /I$(MPIINCLUDE) /Z7 /traceback /debug:full /wrap-margin- $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE_WIN) $(FFLAGS_SUNDIALS_WIN) $(FFLAGS_HDF5_WIN) /Qdiag-disable:10448
impi_intel_win_dv : FCOMPL = $(COMP_FC)
impi_intel_win_dv : obj = fds_impi_intel_win_dv
impi_intel_win_dv : setup_win $(objwin_mpi)
	$(FCOMPL) -o $(obj) $(FFLAGS) /F1000000000 $(objwin_mpi) $(MPILIB) $(LFLAGSMKL_INTEL) $(LFLAGS_HYPRE_WIN) $(LFLAGS_SUNDIALS_WIN) $(LFLAGS_HDF5_WIN) $(LFLAGS_LINK_WIN)

impi_intel_win_openmp_dv : MPILIB = "$(I_MPI_ROOT)\lib\impi.lib"
impi_intel_win_openmp_dv : MPIINCLUDE = "$(I_MPI_ROOT)\include\mpi"
impi_intel_win_openmp_dv : FFLAGS = -D_WIN32 /O1 /I$(MPIINCLUDE) /Z7 /traceback /debug:full /wrap-margin- $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE_WIN) $(FFLAGS_SUNDIALS_WIN) /Qdiag-disable:10448
impi_intel_win_openmp_dv : FOPENMPFLAGS = /Qopenmp
impi_intel_win_openmp_dv : FCOMPL = $(COMP_FC)
impi_intel_win_openmp_dv : obj = fds_impi_intel_win_openmp_dv
impi_intel_win_openmp_dv : setup_win $(objwin_mpi)
	$(FCOMPL) -o $(obj) $(FFLAGS) $(FOPENMPFLAGS) /F1000000000 $(objwin_mpi) $(MPILIB) $(LFLAGSMKL_INTEL_OPENMP) $(LFLAGS_HYPRE_WIN) $(LFLAGS_SUNDIALS_WIN) $(LFLAGS_LINK_WIN)

impi_intel_win_db : MPILIB = "$(I_MPI_ROOT)\lib\impi.lib"
impi_intel_win_db : MPIINCLUDE = "$(I_MPI_ROOT)\include\mpi"
impi_intel_win_db : FFLAGS = -D_WIN32 /Od /nologo /debug:all /I$(MPIINCLUDE) /Z7 /extend-source:132 /warn:unused /warn:nointerfaces /Qtrapuv /fp:strict /fp:except /traceback /check:all /stand:f18 /wrap-margin- $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE_WIN) $(FFLAGS_SUNDIALS_WIN) $(FFLAGS_HDF5_WIN) /Qdiag-disable:10448
impi_intel_win_db : FCOMPL = $(COMP_FC)
impi_intel_win_db : obj = fds_impi_intel_win_db
impi_intel_win_db : setup_win $(objwin_mpi)
	$(FCOMPL) -o $(obj) $(FFLAGS) /F1000000000 $(objwin_mpi) $(MPILIB) $(LFLAGSMKL_INTEL) $(LFLAGS_HYPRE_WIN) $(LFLAGS_SUNDIALS_WIN) $(LFLAGS_HDF5_WIN) $(LFLAGS_LINK_WIN) $(LFLAGS_GPU_WIN)

impi_intel_win_openmp_db : MPILIB = "$(I_MPI_ROOT)\lib\impi.lib"
impi_intel_win_openmp_db : MPIINCLUDE = "$(I_MPI_ROOT)\include\mpi"
impi_intel_win_openmp_db : FFLAGS = -D_WIN32 /Od /nologo /debug:all /I$(MPIINCLUDE) /Z7 /extend-source:132 /warn:unused /warn:nointerfaces /Qtrapuv /fp:strict /fp:except /traceback /check:all /stand:f18 /wrap-margin- $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE_WIN) $(FFLAGS_SUNDIALS_WIN) $(FFLAGS_HDF5_WIN) /Qdiag-disable:10448
impi_intel_win_openmp_db : FOPENMPFLAGS = /Qopenmp_INTEL
impi_intel_win_openmp_db : FCOMPL = $(COMP_FC)
impi_intel_win_openmp_db : obj = fds_impi_intel_win_openmp_db
impi_intel_win_openmp_db : setup_win $(objwin_mpi)
	$(FCOMPL) -o $(obj) $(FFLAGS) $(FOPENMPFLAGS) /F1000000000 $(objwin_mpi) $(MPILIB) $(LFLAGSMKL_INTEL_OPENMP) $(LFLAGS_HYPRE_WIN) $(LFLAGS_SUNDIALS_WIN) $(LFLAGS_HDF5_WIN) $(LFLAGS_LINK_WIN)

# Linux Intel Fortran Compiler and Intel MPI

impi_intel_linux : FFLAGS = $(M64_FLAG) -O2 -ipo -no-wrap-margin $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS) $(FFLAGS_HDF5) -DUSE_IFPORT  -diag-disable=10448
impi_intel_linux : LFLAGSMKL = $(LFLAGSMKL_INTEL) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS) $(LFLAGS_HDF5) $(LFLAGS_GPU)
impi_intel_linux : FCOMPL = $(COMP_FC)
impi_intel_linux : obj = fds_impi_intel_linux
impi_intel_linux : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

impi_intel_linux_openmp : FFLAGS = $(M64_FLAG) -O2 -ipo -no-wrap-margin $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS) $(FFLAGS_HDF5) -DUSE_IFPORT -diag-disable=10448_INTEL
impi_intel_linux_openmp : LFLAGSMKL = $(LFLAGSMKL_INTEL_OPENMP) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS) $(LFLAGS_HDF5)
impi_intel_linux_openmp : FCOMPL = $(COMP_FC)
impi_intel_linux_openmp : FOPENMPFLAGS = -qopenmp
impi_intel_linux_openmp : obj = fds_impi_intel_linux_openmp
impi_intel_linux_openmp : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

impi_intel_linux_db : FFLAGS = $(M64_FLAG) -check all -warn all -diag-error=remark,warn,error -O0 -g -traceback -fpe0 -nofltconsistency -stand:f18 -no-wrap-margin $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS) $(FFLAGS_HDF5) -DUSE_IFPORT -diag-disable=10448_INTEL
impi_intel_linux_db : LFLAGSMKL = $(LFLAGSMKL_INTEL) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS) $(LFLAGS_HDF5) $(LFLAGS_GPU)
impi_intel_linux_db : FCOMPL = $(COMP_FC)
impi_intel_linux_db : obj = fds_impi_intel_linux_db
impi_intel_linux_db : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

impi_intel_linux_openmp_db : FFLAGS = $(M64_FLAG) -check all -warn all -diag-error=remark,warn,error -O0 -g -traceback -fpe0 -nofltconsistency -stand:f18 -no-wrap-margin $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS) $(FFLAGS_HDF5) -DUSE_IFPORT  -diag-disable=10448
impi_intel_linux_openmp_db : LFLAGSMKL = $(LFLAGSMKL_INTEL_OPENMP) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS) $(LFLAGS_HDF5)
impi_intel_linux_openmp_db : FCOMPL = $(COMP_FC)
impi_intel_linux_openmp_db : FOPENMPFLAGS = -qopenmp
impi_intel_linux_openmp_db : obj = fds_impi_intel_linux_openmp_db
impi_intel_linux_openmp_db : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

impi_intel_linux_dv : FFLAGS = $(M64_FLAG)  -warn unused -O1 -g -traceback -no-wrap-margin $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS) $(FFLAGS_HDF5) -DUSE_IFPORT  -diag-disable=10448
impi_intel_linux_dv : LFLAGSMKL = $(LFLAGSMKL_INTEL) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS) $(LFLAGS_HDF5)
impi_intel_linux_dv : FCOMPL = $(COMP_FC)
impi_intel_linux_dv : obj = fds_impi_intel_linux_dv
impi_intel_linux_dv : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

impi_intel_linux_openmp_dv : FFLAGS = $(M64_FLAG) -warn unused -O1 -g -traceback -no-wrap-margin $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS) $(FFLAGS_HDF5) -DUSE_IFPORT  -diag-disable=10448
impi_intel_linux_openmp_dv : LFLAGSMKL = $(LFLAGSMKL_INTEL_OPENMP) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS) $(LFLAGS_HDF5)
impi_intel_linux_openmp_dv : FCOMPL = $(COMP_FC)
impi_intel_linux_openmp_dv : FOPENMPFLAGS = -qopenmp
impi_intel_linux_openmp_dv : obj = fds_impi_intel_linux_openmp_dv
impi_intel_linux_openmp_dv : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

# OSX Intel Fortran and Open MPI

ompi_intel_osx : FFLAGS = $(M64_FLAG) -O2 -stand:f18 -no-wrap-margin $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS)
ompi_intel_osx : LFLAGSMKL = $(LFLAGSMKL_CUSTOM) $(CLT_VERSION) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS)
ompi_intel_osx : LFLAGS = -static-intel
ompi_intel_osx : FCOMPL = $(COMP_FC)
ompi_intel_osx : obj = fds_ompi_intel_osx
ompi_intel_osx : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(LFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_intel_osx_openmp : FFLAGS = $(M64_FLAG) -O2 -stand:f18 -no-wrap-margin $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS)
ompi_intel_osx_openmp : LFLAGSMKL = $(LFLAGSMKL_CUSTOM_OPENMP) $(CLT_VERSION) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS)
ompi_intel_osx_openmp : LFLAGS = -static-intel
ompi_intel_osx_openmp : FOPENMPFLAGS = -qopenmp -qopenmp-link static
ompi_intel_osx_openmp : FCOMPL = $(COMP_FC)
ompi_intel_osx_openmp : obj = fds_ompi_intel_osx_openmp
ompi_intel_osx_openmp : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(LFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_intel_osx_db : FFLAGS = $(M64_FLAG) -check all -fp-stack-check -warn unused -O0 -auto -WB -g -traceback -fpe0 -nofltconsistency -stand:f18 -no-wrap-margin $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS)
ompi_intel_osx_db : LFLAGSMKL = $(LFLAGSMKL_CUSTOM) $(CLT_VERSION) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS)
ompi_intel_osx_db : LFLAGS = -static-intel
ompi_intel_osx_db : FCOMPL = $(COMP_FC)
ompi_intel_osx_db : obj = fds_ompi_intel_osx_db
ompi_intel_osx_db : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(LFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_intel_osx_openmp_db : FFLAGS = $(M64_FLAG) -check all -fp-stack-check -warn unused -O0 -auto -WB -g -traceback -fpe0 -nofltconsistency -stand:f18 -no-wrap-margin $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS)
ompi_intel_osx_openmp_db : LFLAGSMKL = $(LFLAGSMKL_CUSTOM_OPENMP) $(CLT_VERSION) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS)
ompi_intel_osx_openmp_db : LFLAGS = -static-intel
ompi_intel_osx_openmp_db : FOPENMPFLAGS = -qopenmp -qopenmp-link static
ompi_intel_osx_openmp_db : FCOMPL = $(COMP_FC)
ompi_intel_osx_openmp_db : obj = fds_ompi_intel_osx_openmp_db
ompi_intel_osx_openmp_db : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(LFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_intel_osx_dv : FFLAGS = $(M64_FLAG) -warn unused -O1 -g -traceback -stand:f18 -no-wrap-margin  $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS)
ompi_intel_osx_dv : LFLAGSMKL = $(LFLAGSMKL_CUSTOM) $(CLT_VERSION) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS)
ompi_intel_osx_dv : LFLAGS = -static-intel
ompi_intel_osx_dv : FCOMPL = $(COMP_FC)
ompi_intel_osx_dv : obj = fds_ompi_intel_osx_dv
ompi_intel_osx_dv : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(LFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_intel_osx_openmp_dv : FFLAGS = $(M64_FLAG) -warn unused -O1 -g -traceback -stand:f18 -no-wrap-margin  $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS)
ompi_intel_osx_openmp_dv : LFLAGSMKL = $(LFLAGSMKL_CUSTOM_OPENMP) $(CLT_VERSION) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS)
ompi_intel_osx_openmp_dv : LFLAGS = -static-intel
ompi_intel_osx_openmp_dv : FOPENMPFLAGS = -qopenmp -qopenmp-link static
ompi_intel_osx_openmp_dv : FCOMPL = $(COMP_FC)
ompi_intel_osx_openmp_dv : obj = fds_ompi_intel_osx_openmp_dv
ompi_intel_osx_openmp_dv : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(LFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

#*** Intel compiler and OpenMPI in Linux ****

ompi_intel_linux : FFLAGS = $(M64_FLAG) -O2 -ipo -no-wrap-margin $(GITINFO) $(FFLAGSMKL) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS) $(FFLAGS_HDF5) -DUSE_IFPORT  -diag-disable=10448
ompi_intel_linux : LFLAGSMKL = $(LFLAGSMKL_OMPI_INTEL_LINUX) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS) $(LFLAGS_HDF5)
ompi_intel_linux : FCOMPL = $(COMP_FC)
ompi_intel_linux : obj = fds_ompi_intel_linux
ompi_intel_linux : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

#*** GNU Compilers ***

ompi_gnu_linux : FFLAGS = $(M64_FLAG) -O3 -std=f2018 -frecursive -ffpe-summary=none -fall-intrinsics $(GITINFOGNU) $(FFLAGSMKL) $(GFORTRAN_OPTIONS) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS) $(FFLAGS_HDF5)
ompi_gnu_linux : LFLAGSMKL = $(LFLAGSMKL_GNU_OPENMP) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS) $(LFLAGS_HDF5) $(LFLAGS_GPU)
ompi_gnu_linux : FCOMPL = $(COMP_FC)
ompi_gnu_linux : FOPENMPFLAGS = -fopenmp
ompi_gnu_linux : obj = fds_ompi_gnu_linux
ompi_gnu_linux : FFLAGS += -Wno-maybe-uninitialized -Wno-uninitialized -Wno-unused-function
ompi_gnu_linux : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)
# Add ignores for vtk library

ompi_gnu_linux_db : FFLAGS = $(M64_FLAG) -O0 -std=f2018 -ggdb -Wall -Wunused-parameter -Wcharacter-truncation -Wno-target-lifetime -Wno-maybe-uninitialized -Wno-uninitialized -Wno-unused-function -fcheck=all -fbacktrace -ffpe-trap=invalid,zero,overflow -frecursive -ffpe-summary=none -fall-intrinsics -fbounds-check $(GITINFOGNU) $(FFLAGSMKL) $(GFORTRAN_OPTIONS) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS) $(FFLAGS_HDF5)
ompi_gnu_linux_db : LFLAGSMKL = $(LFLAGSMKL_GNU_OPENMP) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS) $(LFLAGS_HDF5) $(LFLAGS_GPU)
ompi_gnu_linux_db : FCOMPL = $(COMP_FC)
ompi_gnu_linux_db : FOPENMPFLAGS = -fopenmp
ompi_gnu_linux_db : obj = fds_ompi_gnu_linux_db
ompi_gnu_linux_db : FFLAGS += -Wno-maybe-uninitialized -Wno-uninitialized -Wno-unused-function
ompi_gnu_linux_db : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_gnu_linux_dv : FFLAGS = $(M64_FLAG) -O1 -fbacktrace -std=f2018 -frecursive -ffpe-summary=none -fall-intrinsics $(GITINFOGNU) $(FFLAGSMKL) $(GFORTRAN_OPTIONS) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS) $(FFLAGS_HDF5)
ompi_gnu_linux_dv : LFLAGSMKL = $(LFLAGSMKL_GNU_OPENMP) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS) $(LFLAGS_HDF5) $(LFLAGS_GPU)
ompi_gnu_linux_dv : FCOMPL = $(COMP_FC)
ompi_gnu_linux_dv : FOPENMPFLAGS = -fopenmp
ompi_gnu_linux_dv : obj = fds_ompi_gnu_linux_dv
ompi_gnu_linux_dv : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_gnu_osx : FFLAGS = $(M64_FLAG) -O2 -std=f2018 -frecursive -ffpe-summary=none -fall-intrinsics $(GITINFOGNU) $(FFLAGSMKL) $(GFORTRAN_OPTIONS) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS) $(FFLAGS_HDF5)
ompi_gnu_osx : LFLAGSMKL = $(LFLAGSMKL_CUSTOM) $(CLT_VERSION) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS) $(LFLAGS_HDF5)
ompi_gnu_osx : FCOMPL = $(COMP_FC)
ompi_gnu_osx : FOPENMPFLAGS = -fopenmp
ompi_gnu_osx : obj = fds_ompi_gnu_osx
ompi_gnu_osx : FFLAGS += -Wno-maybe-uninitialized -Wno-uninitialized -Wno-unused-function
ompi_gnu_osx : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_gnu_osx_db : FFLAGS = $(M64_FLAG) -O0 -std=f2018 -ggdb -Wall -Wcharacter-truncation -Wno-target-lifetime -fcheck=all -fbacktrace -ffpe-trap=invalid,zero,overflow -frecursive -ffpe-summary=none -fall-intrinsics $(GITINFOGNU) $(FFLAGSMKL) $(GFORTRAN_OPTIONS) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS) $(FFLAGS_HDF5)
ompi_gnu_osx_db : LFLAGSMKL = $(LFLAGSMKL_CUSTOM) $(CLT_VERSION) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS) $(LFLAGS_HDF5)
ompi_gnu_osx_db : FCOMPL = $(COMP_FC)
ompi_gnu_osx_db : FOPENMPFLAGS = -fopenmp
ompi_gnu_osx_db : obj = fds_ompi_gnu_osx_db
ompi_gnu_osx_db : FFLAGS += -Wno-maybe-uninitialized -Wno-uninitialized -Wno-unused-function
ompi_gnu_osx_db : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_gnu_osx_dv : FFLAGS = $(M64_FLAG) -O1 -fbacktrace -std=f2018 -frecursive -ffpe-summary=none -fall-intrinsics $(GITINFOGNU) $(FFLAGSMKL) $(GFORTRAN_OPTIONS) $(FFLAGS_HYPRE) $(FFLAGS_SUNDIALS) $(FFLAGS_HDF5)
ompi_gnu_osx_dv : LFLAGSMKL = $(LFLAGSMKL_CUSTOM) $(CLT_VERSION) $(LFLAGS_HYPRE) $(LFLAGS_SUNDIALS) $(LFLAGS_HDF5)
ompi_gnu_osx_dv : FCOMPL = $(COMP_FC)
ompi_gnu_osx_dv : FOPENMPFLAGS = -fopenmp
ompi_gnu_osx_dv : obj = fds_ompi_gnu_osx_dv
ompi_gnu_osx_dv : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)


# *** Object Dependencies ***

prec.o :
imkl.o : prec.o
cons.o : prec.o
chem.o : cons.o func.o
prop.o : prec.o
devc.o : prec.o
type.o : prec.o cons.o imkl.o
data.o : prec.o func.o cons.o prop.o
rcal.o : func.o prec.o cons.o
mesh.o : prec.o type.o
func.o : prec.o prop.o cons.o type.o mesh.o devc.o
gsmv.o : prec.o func.o cons.o mesh.o
smvv.o : geom.o prec.o func.o cons.o data.o
turb.o : prec.o func.o cons.o mesh.o
soot.o : prec.o mesh.o cons.o func.o turb.o
pois.o : prec.o cons.o
geom.o : gsmv.o func.o prec.o cons.o type.o mesh.o turb.o imkl.o data.o
ccib.o : gsmv.o func.o prec.o cons.o type.o mesh.o turb.o geom.o
radi.o : func.o prec.o cons.o type.o mesh.o rcal.o geom.o
part.o : func.o prec.o cons.o type.o mesh.o devc.o data.o geom.o soot.o ccib.o
vege.o : func.o prec.o cons.o type.o mesh.o geom.o
ctrl.o : prec.o cons.o type.o mesh.o func.o
hvac.o : func.o prec.o cons.o type.o mesh.o ctrl.o data.o geom.o
mass.o : func.o prec.o cons.o type.o mesh.o turb.o soot.o geom.o ccib.o
wall.o : func.o prec.o cons.o type.o mesh.o hvac.o mass.o soot.o geom.o ccib.o
fire.o : func.o prec.o cons.o type.o mesh.o mass.o soot.o radi.o chem.o
velo.o : func.o prec.o cons.o type.o mesh.o turb.o geom.o ccib.o
pres.o : func.o prec.o cons.o type.o mesh.o pois.o velo.o geom.o ccib.o imkl.o
init.o : func.o prec.o cons.o type.o mesh.o devc.o radi.o pois.o data.o pres.o geom.o
vtkf.o : func.o prec.o cons.o type.o mesh.o data.o geom.o
dump.o : func.o prec.o cons.o type.o mesh.o devc.o smvv.o turb.o data.o radi.o soot.o geom.o ccib.o fire.o vtkf.o
read.o : func.o prec.o cons.o type.o mesh.o devc.o ctrl.o radi.o hvac.o data.o geom.o mass.o soot.o prop.o
divg.o : func.o prec.o cons.o type.o mesh.o mass.o ccib.o
main.o : func.o prec.o cons.o type.o mesh.o devc.o smvv.o mass.o divg.o velo.o wall.o fire.o radi.o part.o vege.o dump.o read.o init.o pres.o pois.o ctrl.o turb.o hvac.o data.o geom.o ccib.o soot.o prop.o chem.o vtkf.o

prec.obj :
imkl.obj : prec.obj
cons.obj : prec.obj
chem.obj : cons.obj func.obj
prop.obj : prec.obj
devc.obj : prec.obj
type.obj : prec.obj cons.obj imkl.obj
data.obj : prec.obj func.obj cons.obj prop.obj
rcal.obj : func.obj prec.obj cons.obj
mesh.obj : prec.obj type.obj
func.obj : prec.obj prop.obj cons.obj type.obj mesh.obj devc.obj
gsmv.obj : prec.obj func.obj cons.obj mesh.obj
smvv.obj : geom.obj prec.obj func.obj cons.obj data.obj
turb.obj : prec.obj func.obj cons.obj mesh.obj
soot.obj : prec.obj mesh.obj cons.obj func.obj turb.obj
pois.obj : prec.obj cons.obj
geom.obj : gsmv.obj func.obj prec.obj cons.obj type.obj mesh.obj turb.obj imkl.obj data.obj
ccib.obj : gsmv.obj func.obj prec.obj cons.obj type.obj mesh.obj turb.obj geom.obj
radi.obj : func.obj prec.obj cons.obj type.obj mesh.obj rcal.obj geom.obj
part.obj : func.obj prec.obj cons.obj type.obj mesh.obj devc.obj data.obj geom.obj soot.obj ccib.obj
vege.obj : func.obj prec.obj cons.obj type.obj mesh.obj geom.obj
ctrl.obj : prec.obj cons.obj type.obj mesh.obj func.obj
hvac.obj : func.obj prec.obj cons.obj type.obj mesh.obj ctrl.obj data.obj geom.obj
mass.obj : func.obj prec.obj cons.obj type.obj mesh.obj turb.obj soot.obj geom.obj ccib.obj
wall.obj : func.obj prec.obj cons.obj type.obj mesh.obj hvac.obj mass.obj soot.obj geom.obj ccib.obj
fire.obj : func.obj prec.obj cons.obj type.obj mesh.obj mass.obj soot.obj radi.obj chem.obj
velo.obj : func.obj prec.obj cons.obj type.obj mesh.obj turb.obj geom.obj ccib.obj
pres.obj : func.obj prec.obj cons.obj type.obj mesh.obj pois.obj velo.obj geom.obj ccib.obj imkl.obj
init.obj : func.obj prec.obj cons.obj type.obj mesh.obj devc.obj pois.obj data.obj pres.obj geom.obj
vtkf.obj  : func.obj prec.obj cons.obj type.obj mesh.obj data.obj
dump.obj : func.obj prec.obj cons.obj type.obj mesh.obj devc.obj smvv.obj turb.obj data.obj radi.obj soot.obj geom.obj ccib.obj fire.obj vtkf.obj
read.obj : func.obj prec.obj cons.obj type.obj mesh.obj devc.obj ctrl.obj radi.obj hvac.obj data.obj geom.obj mass.obj soot.obj prop.obj
divg.obj : func.obj prec.obj cons.obj type.obj mesh.obj mass.obj ccib.obj
main.obj : func.obj prec.obj cons.obj type.obj mesh.obj devc.obj smvv.obj mass.obj divg.obj velo.obj wall.obj fire.obj radi.obj part.obj vege.obj dump.obj read.obj init.obj pres.obj pois.obj ctrl.obj turb.obj hvac.obj data.obj geom.obj ccib.obj soot.obj prop.obj chem.obj vtkf.obj

#*** Clean Target to remove Object and Module files ***

.PHONY : clean
clean:
	-rm -f *.o *.mod *.obj

.PHONY : winclean
winclean:
	-erase *.mod *.obj
