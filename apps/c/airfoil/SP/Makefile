#
# The following environment variables should be predefined:
#
# CUDA_INSTALL_PATH
# PARMETIS_INSTALL_PATH
# PTSCOTCH_INSTALL_PATH
# HDF5_INSTALL_PATH
#
# OP2_INSTALL_PATH
# OP2_COMPILER (gnu,intel,etc)
#

#
# set paths for header files and libraries
#
OP2_INC		= -I$(OP2_INSTALL_PATH)/c/include
OP2_LIB		= -L$(OP2_INSTALL_PATH)/c/lib

CUDA_INC	= -I$(CUDA_INSTALL_PATH)/include
CUDA_LIB	= -L$(CUDA_INSTALL_PATH)/lib64


ifeq ($(OP2_COMPILER),gnu)
  CPP		= g++
  CPPFLAGS	= -g -fPIC -DUNIX -Wall
#  CPPFLAGS	= -O3 -fPIC -DUNIX -Wall -Wextra
  OMPFLAGS	= -fopenmp
  MPICPP	= /usr/bin/mpiCC
  MPIFLAGS	= $(CCFLAGS)
else
ifeq ($(OP2_COMPILER),intel)
  CPP		= icpc
  CCFLAGS	= -O2 -vec-report -xSSE2,SSE3,SSE4.1,SSE4.2 -g #-DCOMM_PERF #-DDEBUG
  OMPFLAGS	= -openmp -openmp-report2
  MPICPP	= mpiCC
  MPIFLAGS	= $(CCFLAGS)
else
print:
	@echo "unrecognised value for OP2_COMPILER"
endif
endif

#
# set flags for NVCC compilation and linking
#
NVCCFLAGS	= -O3 -arch=sm_20 -Xptxas=-v -use_fast_math
#NVCCFLAGS	= -g -G -arch=sm_20 -Xptxas=-v -use_fast_math


#
# partitioning software for MPI versions
#

PARMETIS_INC 	= -I$(PARMETIS_INSTALL_PATH) -DPARMETIS
PARMETIS_LIB 	= -L$(PARMETIS_INSTALL_PATH) -lparmetis \
	       	  -L$(PARMETIS_INSTALL_PATH) -lmetis

PTSCOTCH_INC 	= -I$(PTSCOTCH_INSTALL_PATH)/include -DPTSCOTCH
PTSCOTCH_LIB 	= -L$(PTSCOTCH_INSTALL_PATH)/lib/ -lptscotch \
                  -L$(PTSCOTCH_INSTALL_PATH)/lib/ -lptscotcherr

#
# master to make all versions
#

all: airfoil_seq airfoil_openmp airfoil_cuda airfoil_mpi

#
# simple sequential version
#

airfoil_seq: airfoil.cpp save_soln.h adt_calc.h res_calc.h bres_calc.h Makefile
	     $(CPP) airfoil.cpp $(OP2_INC) $(OP2_LIB) -lop2_seq -o airfoil_seq

#
# x86 version using kernel files generated by op2.m
#

airfoil_openmp:	airfoil_op.cpp airfoil_kernels.cpp \
		save_soln_kernel.cpp  save_soln.h \
		adt_calc_kernel.cpp   adt_calc.h  \
		res_calc_kernel.cpp   res_calc.h  \
		bres_calc_kernel.cpp  bres_calc.h \
		update_kernel.cpp     update.h    \
                Makefile
		$(CPP) $(CCFLAGS) $(OMPFLAGS) $(OP2_INC) $(OP2_LIB) \
		airfoil_op.cpp airfoil_kernels.cpp -lm -lop2_openmp -o airfoil_openmp

#
# CUDA version using kernel files generated by op2.m
#

airfoil_cuda:	airfoil_op.cpp airfoil_kernels_cu.o Makefile 
		$(CPP) $(CPPFLAGS) $(CUDA_INC) $(OP2_INC) $(OP2_LIB) $(CUDA_LIB) \
		airfoil_op.cpp airfoil_kernels_cu.o -lcudart -lop2_cuda -o airfoil_cuda

airfoil_kernels_cu.o:	airfoil_kernels.cu      \
		save_soln_kernel.cu save_soln.h \
		adt_calc_kernel.cu  adt_calc.h  \
		res_calc_kernel.cu  res_calc.h  \
		bres_calc_kernel.cu bres_calc.h \
		update_kernel.cu    update.h    \
                Makefile
		nvcc  $(INC) $(NVCCFLAGS) $(OP2_INC) \
		-c -o airfoil_kernels_cu.o \
		airfoil_kernels.cu

#
# mpi with sequential-nodes version
#

airfoil_mpi: airfoil_mpi.cpp save_soln.h adt_calc.h res_calc.h bres_calc.h Makefile 
	$(MPICPP) $(MPIFLAGS) airfoil_mpi.cpp $(OP2_INC) $(PARMETIS_INC) $(PTSCOTCH_INC) \
	$(OP2_LIB) -lop2_mpi $(PARMETIS_LIB) $(PTSCOTCH_LIB) -o airfoil_mpi

#
# cleanup
#

clean:
		rm -f airfoil_seq airfoil_openmp airfoil_cuda airfoil_mpi *.o
