# Makefile for building spmv example (single GPU and multi GPU) using AMGX C++ code directly 

# System paths
CC = /usr/bin/cc
CPP = /usr/bin/c++
CUDA_PATH = /usr/local/cuda
MPI_PATH = /opt/openmpi-1.10.2
NVCC = $(CUDA_PATH)/bin/nvcc
ARCH = --gpu-architecture=compute_35 --gpu-code=compute_35,sm_35

NVCC_FLAGS = -std=c++11 $(ARCH) -Xcompiler ,\"-static-libgcc\",\"-fopenmp\",\"-DRAPIDJSON_DEFINED\",\"-O3\",\"-DNDEBUG\" -Xcompiler=-rdynamic -Xcompiler=-fPIC -Xcompiler=-fvisibility=default -DCUDA90_COMPILER -O3 -DNDEBUG --Werror cross-execution-space-call -DNVCC
NVCC_D_FLAGS = -Xcompiler ,\"-DAMGX_WITH_MPI\"
NVCC_L_FLAGS = $(ARCH)
MPI_L_FLAGS = -lmpi -L$(MPI_PATH)/lib
EXTRA_LIBS = -lcublas -lcusparse -lcusolver -Xlinker=-rpath=$(CUDA_PATH)/lib64
AMGX_ROOT = ../..
AMGX_INCLUDE = -I$(AMGX_ROOT)/../../thrust -I$(AMGX_ROOT)/base/include -I$(CUDA_PATH)/include -I$(AMGX_ROOT)/external/rapidjson/include -I$(MPI_PATH)/include

BASE_CU_FILES := ../../base/src/misc.cu ../../base/src/device_properties.cu ../../base/src/logger.cu ../../base/src/auxdata.cu ../../base/src/amgx_cusparse.cu ../../base/src/amgx_cublas.cu ../../base/src/amg_config.cu ../../base/src/global_thread_handle.cu ../../base/src/error.cu ../../base/src/thread_manager.cu ../../base/src/resources.cu ../../base/src/matrix.cu ../../base/src/multiply.cu ../../base/src/hash_workspace.cu ../../base/src/csr_multiply.cu ../../base/src/csr_multiply_sm20.cu ../../base/src/csr_multiply_sm35.cu ../../base/src/matrix_coloring/matrix_coloring.cu ../../base/src/distributed/distributed_manager.cu ../../base/src/distributed/distributed_arranger.cu ../../base/src/distributed/comms_visitors1.cu ../../base/src/distributed/comms_visitors2.cu ../../base/src/distributed/comms_visitors3.cu ../../base/src/distributed/comms_visitors4.cu ../../base/src/distributed/comms_mpi_hostbuffer_stream.cu ../../base/src/distributed/comms_mpi_gpudirect.cu ../../base/src/distributed/distributed_comms.cu

OBJDIR_S_GPU := obj_s
OBJDIR_M_GPU := obj_m

OBJS_S := $(BASE_CU_FILES:%.cu=%_s.o)
OBJS_M := $(BASE_CU_FILES:%.cu=%_m.o)


%_s.o: %.cu
	mkdir -p $(OBJDIR_S_GPU); $(NVCC) $(NVCC_FLAGS) $(AMGX_INCLUDE) -c $< -o $(OBJDIR_S_GPU)/$(notdir $@)

%_m.o: %.cu
	mkdir -p $(OBJDIR_M_GPU); $(NVCC) $(NVCC_FLAGS) $(NVCC_D_FLAGS) $(AMGX_INCLUDE) -c $< -o $(OBJDIR_M_GPU)/$(notdir $@)

example: $(OBJS_S)
	$(NVCC) $(NVCC_FLAGS) $(AMGX_INCLUDE) -c amgx_spmv_internal.cu -o $(OBJDIR_S_GPU)/amgx_spmv_internal.o
	$(NVCC) $(NVCC_L_FLAGS) $(EXTRA_LIBS) $(OBJDIR_S_GPU)/*.o -o amgx_spmv

example_distributed: $(OBJS_M)
	$(NVCC) $(NVCC_FLAGS) $(NVCC_D_FLAGS) $(AMGX_INCLUDE) -c amgx_spmv_distributed_internal.cu -o $(OBJDIR_M_GPU)/amgx_spmv_distributed_internal.o
	$(NVCC) $(NVCC_L_FLAGS) $(MPI_L_FLAGS) $(EXTRA_LIBS) $(OBJDIR_M_GPU)/*.o -o amgx_spmv_distributed