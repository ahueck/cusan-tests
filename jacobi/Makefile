# Copyright (c) 1993-2015, NVIDIA CORPORATION. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#  * Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
#  * Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in the
#    documentation and/or other materials provided with the distribution.
#  * Neither the name of NVIDIA CORPORATION nor the names of its
#    contributors may be used to endorse or promote products derived
#    from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS ``AS IS'' AND ANY
# EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
# PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL THE COPYRIGHT OWNER OR
# CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
# EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
# PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
# PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY
# OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

# Compilers
MPICC=$(PREP) ${CUSAN_MPICC_EXEC}
MPILD=$(PREP) ${CUSAN_MPICXX_EXEC}
NVCC=$(PREP) ${CUSAN_MPICXX_EXEC}

# Flags
CFLAGS=-std=c99 -O3 -march=native -Wall -g ${CUTEST_ADDITIONAL_CFLAGS}
MPICFLAGS=-I${MPI_HOME}/include
CUDACFLAGS=-I${CUDA_INSTALL_PATH}/include

CUDALDFLAGS=-L${CUDA_INSTALL_PATH}/lib64 -lcudart ${CUTEST_ADDITIONAL_CFLAGS}

# Description of binaries
BINDIR=./bin
JACOBI_CUDA_NORMAL_MPI=$(BINDIR)/jacobi_cuda_normal_mpi
JACOBI_CUDA_AWARE_MPI=$(BINDIR)/jacobi_cuda_aware_mpi
BINARIES=$(JACOBI_CUDA_NORMAL_MPI) $(JACOBI_CUDA_AWARE_MPI)
BINARIES_VANILLA=$(JACOBI_CUDA_NORMAL_MPI)_vanilla $(JACOBI_CUDA_AWARE_MPI)_vanilla
BINARIES_TSAN_VANILLA=$(JACOBI_CUDA_NORMAL_MPI)_tsan_vanilla $(JACOBI_CUDA_AWARE_MPI)_tsan_vanilla
BINARIES_CUSAN=$(JACOBI_CUDA_NORMAL_MPI)_cusan $(JACOBI_CUDA_AWARE_MPI)_cusan

# Commands
all: $(BINARIES)

device.o: Jacobi.h Device.cu Makefile
	$(NVCC) $(MPICFLAGS) $(CUDACFLAGS) --cuda-gpu-arch=sm_70 -c Device.cu -o device.o

input.o: Jacobi.h Input.c Makefile
	$(MPICC) $(MPICFLAGS) $(CUDACFLAGS) $(CFLAGS) -c Input.c -o input.o

jacobi.o: Jacobi.h Jacobi.c Makefile
	$(MPICC) $(MPICFLAGS) $(CUDACFLAGS) $(CFLAGS) -c Jacobi.c -o jacobi.o

host.o: Jacobi.h Host.c Makefile
	$(MPICC) $(MPICFLAGS) $(CFLAGS) $(CUDACFLAGS) -c Host.c -o host.o

cuda_normal_mpi.o: Jacobi.h CUDA_Normal_MPI.c Makefile
	$(MPICC) $(MPICFLAGS) $(CFLAGS) $(CUDACFLAGS) -c CUDA_Normal_MPI.c -o cuda_normal_mpi.o

cuda_aware_mpi.o: Jacobi.h CUDA_Aware_MPI.c Makefile
	$(MPICC) $(MPICFLAGS) $(CFLAGS) $(CUDACFLAGS) -c CUDA_Aware_MPI.c -o cuda_aware_mpi.o

$(JACOBI_CUDA_NORMAL_MPI): jacobi.o input.o host.o device.o cuda_normal_mpi.o Makefile
	mkdir -p $(BINDIR)
	$(MPILD) -o $(JACOBI_CUDA_NORMAL_MPI) jacobi.o input.o host.o device.o  cuda_normal_mpi.o $(CUDALDFLAGS)
	
$(JACOBI_CUDA_AWARE_MPI): jacobi.o input.o host.o device.o cuda_aware_mpi.o Makefile
	mkdir -p $(BINDIR)
	$(MPILD) -o $(JACOBI_CUDA_AWARE_MPI) jacobi.o input.o host.o device.o cuda_aware_mpi.o $(CUDALDFLAGS)

doc: CUDA_Normal_MPI.c CUDA_Aware_MPI.c Device.cu Host.c Input.c Makefile Jacobi.c Jacobi.h Jacobi.doxygen
	doxygen Jacobi.doxygen
	
clean:
	rm -rf *.o *~ *.fatbin *.ptx

veryclean:
	$(BINARIES) $(BINARIES_VANILLA) $(BINARIES_TSAN_VANILLA) $(BINARIES_CUSAN) *.yaml
