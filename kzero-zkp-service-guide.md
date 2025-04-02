# Guide to Generating Witness and Proof for Kzero Circuit Using Kzero ZKP Witness & ZKP Service

## Introduction
This guide provides a step-by-step process for generating a witness and proof with Kzero. The guide covers circuit compilation, witness generation, and proof generation using GPU acceleration.

## Prerequisites
Before proceeding, ensure that you have the following installed:
- [Circom](https://github.com/iden3/circom) for compiling circuits
- [kZero Circuit](https://github.com/kzero-xyz/kzero-circuit) repository
- A C++ compiler (e.g., `g++` or `clang++`)
- A compatible[ GPU ZKP Generation Service](https://github.com/kzero-xyz/rapidsnark) (for GPU-accelerated proof generation)
## Witness Service
### Step 1: Compile the Circuit & Generate Cpp
First, navigate to the kZero circuit repository and compile the circuit using Circom:

```sh
circom zkLogin.circom --r1cs --sym --c
```
The `--c` flag ensures that the compilation generates C++ files, which are required for further processing.

> `--c`: It generates the directory zkLogin_cpp that contains several files (zkLogin.cpp, zkLogin.dat, and other common files for every compiled program like main.cpp, MakeFile, etc) needed to compile the C++ code to generate the witness.

### Step 2: Build the C++ Code
Once the C++ files are generated, use make to compile them into an executable binary:

```sh
cd zkLogin_cpp
make
```
This process will produce a executable binary called zkLogin_cpp that can be used to generate the witness.

> Note: To compile the C++ source, we rely on some libraries that you need to have installed in your system. In particular, we use `nlohmann-json3-dev`, `libgmp-dev` and `nasm`.


### Step 3: Generate the Witness
After the executable is created, we execute it indicating the input file and the name for the witness file:
```sh
./zkLogin input.json witness.wtns
```
This command processes the input JSON and outputs the witness in witness.wtns.

## ZKP Generation Service(GPU)

After generating the witness, we can use the standard rapidsnark to directly generate the zkproof. However, for service use, the efficiency is relatively low. After researching various ZKP generation methods, we chose the [GPU version](https://github.com/kzero-xyz/rapidsnark) of rapidsnark for ZKP generation. This version significantly accelerates the proof generation process by leveraging GPU capabilities.

### 1. Prerequisites
Before using the GPU version of rapidsnark, ensure that your environment meets the following prerequisites:

- **CUDA Toolkit**: Make sure you have the appropriate CUDA Toolkit installed. The required version may depend on your GPU.
- **CMake**: Required for building the project.
- **GCC**: Ensure you have the necessary GCC version installed for compiling the code.
- **Ubuntu 22.04**: This guide assumes you are using Ubuntu 22.04 as your operating system.
- **x86_64 Host Machine**: The GPU version of rapidsnark is designed for 64-bit architecture.

For more detailed prerequisites, please refer to the [ICICLE repository](https://github.com/ingonyama-zk/icicle?tab=readme-ov-file#prerequisites).

### 2. Dependencies
To install the required dependencies for building rapidsnark, run the following command:

```bash
sudo apt-get install build-essential libgmp-dev libsodium-dev nasm curl m4
```
This will ensure that you have all the necessary libraries and tools to build the project.

### 3. Compile the Prover
1. First, initialize and update the submodules for the project:
```bash=
git submodule init
git submodule update
```
2. Next, run the build script to prepare the GMP library for the host machine:
```bash=
./build_gmp.sh host
```
3. Create a build directory, navigate to it, and configure the project using cmake:
```bash=
mkdir build_prover && cd build_prover
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=../package
```
4. Finally, build and install the prover:
```bash=
make -j$(nproc) && make install
```
The -j$(nproc) flag ensures that the build process utilizes all available cores for faster compilation.

### 4. Building the Proof
Once you have successfully built the prover, you can use it to generate the proof. The command to run the prover is as follows:
```bash=
./package/bin/prover_cuda <circuit.zkey> <witness.wtns> <proof.json> <public.json>
```
- `circuit.zkey`: The .zkey file generated during the circuit setup.
- `witness.wtns`: The witness file generated in the previous steps.
- `proof.json`: The output file that will contain the generated zk-proof.
- `public.json`: The output file that will contain the public inputs.

Once the proof is generated, you will have a `proof.json` file that contains the zero-knowledge proof and a `public.json` file that holds the public inputs.

## Conclusion
Using the GPU version of rapidsnark allows for much faster proof generation, making it a more efficient solution for services requiring large-scale zk-proof generation. With the setup & usage steps above, you can now build and use the GPU-accelerated prover to generate zk-proofs for Kzero based application.

For more details and troubleshooting, you can refer to the [Kzero-Rapidsnark-GPU](https://github.com/kzero-xyz/rapidsnark) repository.
