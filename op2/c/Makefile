SRC = src
INC = include
LIB = lib
OBJ = obj
CUDA_INC = $(CUDA)/include

CC = gcc -g
CPP = g++ -g
NVCC = nvcc -g

FLAGS = -Wall

.PHONY: clean

all: core core_nvcc reference runtime runtime_nvcc cuda

core: $(INC)/op_lib_core.h $(SRC)/op_lib_core.cpp
	$(CPP) $(FLAGS) -I$(INC) -c $(SRC)/op_lib_core.cpp -o $(OBJ)/op_lib_core.o
#	ar -r $(LIB)/libop2_core.a $(OBJ)/op_lib_core.o

core_nvcc: $(INC)/op_lib_core.h $(SRC)/op_lib_core.cpp
	$(NVCC) -I$(INC) -c $(SRC)/op_lib_core.cpp -o $(OBJ)/op_lib_core_nvcc.o
	ar -r $(LIB)/libop2_core_nvcc.a $(OBJ)/op_lib_core_nvcc.o

reference: $(INC)/op_reference.h $(SRC)/op_reference.cpp $(OBJ)/op_lib_core.o
	$(CPP) $(FLAGS) -I$(INC) -c $(SRC)/op_reference.cpp -o $(OBJ)/op_reference.o
	ar -r $(LIB)/libop2_reference.a $(OBJ)/op_reference.o $(OBJ)/op_lib_core.o

runtime: $(INC)/op_rt_support.h $(SRC)/op_rt_support.cpp $(INC)/op_lib_core.h $(SRC)/op_lib_core.cpp
	$(CPP) $(FLAGS) -I$(INC) -c $(SRC)/op_rt_support.cpp -o $(OBJ)/op_rt_support.o
	ar -r $(LIB)/libop2_rt_support.a $(OBJ)/op_rt_support.o

runtime_nvcc: $(INC)/op_rt_support.h $(SRC)/op_rt_support.cpp $(INC)/op_lib_core.h $(SRC)/op_lib_core.cpp
	$(NVCC) --compiler-options $(FLAGS) -I$(INC) -c $(SRC)/op_rt_support.cpp -o $(OBJ)/op_rt_support.o
	ar -r $(LIB)/libop2_rt_support_nvcc.a $(OBJ)/op_rt_support.o

cuda: $(INC)/op_cuda_rt_support.h $(SRC)/cuda/op_cuda_decl.cpp $(SRC)/cuda/op_cuda_rt_support.cpp $(OBJ)/op_lib_core.o
	$(CPP) -I$(CUDA_INSTALL_PATH)/include  -I$(INC) -c $(SRC)/cuda/op_cuda_decl.cpp -o $(OBJ)/cuda/op_cuda_decl.o
	$(CPP) -I$(CUDA_INSTALL_PATH)/include -I$(INC) -c $(SRC)/cuda/op_cuda_rt_support.cpp -o $(OBJ)/cuda/op_cuda_rt_support.o
	ar -r $(LIB)/libop2_cuda.a $(OBJ)/cuda/op_cuda_rt_support.o $(OBJ)/cuda/op_cuda_decl.o $(OBJ)/op_lib_core.o

openmp: $(SRC)/openmp/op_openmp_decl.cpp $(OBJ)/op_lib_core.o
	$(CPP) -I$(INC) -c $(SRC)/openmp/op_openmp_decl.cpp -o $(OBJ)/openmp/op_openmp_decl.o
	ar -r $(LIB)/libop2_openmp.a $(OBJ)/openmp/op_openmp_decl.o $(OBJ)/op_lib_core.o

clean:
	\rm -if $(OBJ)/*.o
	\rm -if $(OBJ)/cuda/*.o
	\rm -if $(OBJ)/openmp/*.o
	\rm -if $(LIB)/*.a
	\rm -if ./*~