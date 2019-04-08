This gives the steps to build Tensorflow 2.0 for the CPU partitions on LISA/Cartesius. It can be reused for other systems.

If all steps are followed in sequence, you should be able to obtain a working Tensoflow installation. If needed, remember to add the ```-g``` flag to the ```bazel build``` step. This is tested for Python 3.7, however it should work the same for any Python 2.7 stack.

# Load pre-requisites
```bash 
module load GCCcore/7.3.0
module load Java/1.8.0_152
module load Python/3.7.0-intel-2018b
module load hwloc/1.11.10-GCCcore-7.3.0
```
Loading this EasyBuild Python module and our wrappers will set:
```mpicc``` to ```mpigcc for the Intel(R) MPI Library 2018 Update 3 for Linux*```
```mpicxx``` to ```mpigxx for the Intel(R) MPI Library 2018 Update 3 for Linux*```

```bash
module list
Currently Loaded Modulefiles:
 1) libgfortran/32/1(default)                           18) iimpi/2018b                      
 2) stdenv/1.3(default)                                 19) imkl/2018.3.222-iimpi-2018b      
 3) licenses/1.0(default)                               20) intel/2018b                      
 4) oldwheezy/1.0(default)                              21) bzip2/1.0.6-GCCcore-7.3.0        
 5) torque/default                                      22) zlib/1.2.11-GCCcore-7.3.0        
 6) slurm-tools                                         23) ncurses/6.1-GCCcore-7.3.0        
 7) surfsara/1.1(default)                               24) libreadline/7.0-GCCcore-7.3.0    
 8) EasyBuild/3.8.0                                     25) Tcl/8.6.8-GCCcore-7.3.0          
 9) compilerwrappers                                    26) SQLite/3.24.0-GCCcore-7.3.0      
10) eb/3.8.0(default)                                   27) XZ/5.2.4-GCCcore-7.3.0           
11) GCCcore/7.3.0                                       28) GMP/6.1.2-GCCcore-7.3.0          
12) Java/1.8.0_152                                      29) libffi/3.2.1-GCCcore-7.3.0       
13) binutils/2.30-GCCcore-7.3.0                         30) Python/3.7.0-intel-2018b         
14) icc/2018.3.222-GCC-7.3.0-2.30                       31) numactl/2.0.11-GCCcore-7.3.0     
15) ifort/2018.3.222-GCC-7.3.0-2.30                     32) libxml2/2.9.8-GCCcore-7.3.0      
16) iccifort/2018.3.222-GCC-7.3.0-2.30                  33) libpciaccess/0.14-GCCcore-7.3.0  
17) impi/2018.3.222-iccifort-2018.3.222-GCC-7.3.0-2.30  34) hwloc/1.11.10-GCCcore-7.3.0 
```

If you are missing the 2018b suite, you can use any other GCC 5+ toolchain. On LISA/Cartesius you can install this Python version and all dependencies with:
```bash
module load eb
eblocalinstall Python-3.7.0-intel-2018b.eb --robot
```
# Setup Bazel

### Download Bazel (latest version)
You want to download the "-dist" i.e bazel-0.24.0-dist.zip from https://github.com/bazelbuild/bazel/releases/download/0.24.0/bazel-0.24.0-dist.zip

Please note that for the current 2.0 alpha version you want Bazel 0.24.0 not 0.24.1 !

### Build Bazel
```bash
wget https://github.com/bazelbuild/bazel/releases/download/0.24.0/bazel-0.24.0-dist.zip
unzip bazel-0.24.0-dist.zip -d bazel
cd bazel
``` 
according to https://docs.bazel.build/versions/master/install-compile-source.html

Then build Bazel from source, with all dependencies e.g. protobuf:
```bash
env EXTRA_BAZEL_ARGS="--host_javabase=@local_jdk//:jdk" bash ./compile.sh
```

### Check Bazel version
```bash
./output/bazel version
```
Example output:
```bash
WARNING: Output base '/home/damian/.cache/bazel/_bazel_damian/36c72147563d5a46d694a11cb3ab7984' is on NFS. This may lead to surprising failures and undetermined behavior.
Extracting Bazel installation...
WARNING: ignoring _JAVA_OPTIONS in environment.
Starting local Bazel server and connecting to it...
Build label: 0.24.0- (@non-git)
Build target: bazel-out/k8-opt/bin/src/main/java/com/google/devtools/build/lib/bazel/BazelServer_deploy.jar
Build time: Mon Apr 8 13:30:41 2019 (1554730241)
Build timestamp: 1554730241
Build timestamp as int: 1554730241
```

### Set Bazel in $PATH
```bash
mkdir -p ${HOME}/install/bin
cp output/bazel ${HOME}/install/bin
export PATH=${HOME}/install/bin:$PATH
```

# Setup Tensorflow 2.0

### Download Tensorflow
Tensorflow 2.0 home: https://github.com/tensorflow/tensorflow/tree/r2.0

Official build howto: https://www.tensorflow.org/install/source

Intel build howto: https://software.intel.com/en-us/articles/intel-optimization-for-tensorflow-installation-guide


```bash 
git clone https://github.com/tensorflow/tensorflow.git
cd tensorflow
git checkout r2.0
``` 

### Patch TF2.0 for hwloc error
https://github.com/bazelbuild/bazel/issues/6461

Error is:
```bash
ERROR: /home/damian/.cache/bazel/_bazel_damian/9fcb7b127fd29ce682f779e62c1e54f6/external/hwloc/BUILD.bazel:189:54: The `+` operator for dicts is deprecated and no longer supported. Please use the `update` method instead. You can temporarily enable the `+` operator by passing the flag --incompatible_disallow_dict_plus=false
ERROR: /home/damian/.cache/bazel/_bazel_damian/9fcb7b127fd29ce682f779e62c1e54f6/external/hwloc/BUILD.bazel:200:9: Traceback (most recent call last):
	File "/home/damian/.cache/bazel/_bazel_damian/9fcb7b127fd29ce682f779e62c1e54f6/external/hwloc/BUILD.bazel", line 195
		template_rule(name = "include_private_hwloc_au...", <3 more arguments>)
	File "/home/damian/.cache/bazel/_bazel_damian/9fcb7b127fd29ce682f779e62c1e54f6/external/hwloc/BUILD.bazel", line 199, in template_rule
		if_cuda(_INCLUDE_PRIVATE_HWLOC_AUTOIGEN_..., ...)
	File "/home/damian/.cache/bazel/_bazel_damian/9fcb7b127fd29ce682f779e62c1e54f6/external/hwloc/BUILD.bazel", line 200, in if_cuda
		_INCLUDE_PRIVATE_HWLOC_AUTOIGEN_CONFIG_H_CUDA_SUBS
name '_INCLUDE_PRIVATE_HWLOC_AUTOIGEN_CONFIG_H_CUDA_SUBS' is not defined
```

Solution:
```bash
git cherry-pick 21425b41215989956acc3b06cee506e9c6ac57f0
```

### Install Python depencies in your current environment
```bash
pip install pip six numpy wheel setuptools mock --user
pip install keras_applications keras_preprocessing --user
```

### Build the TF2.0 Python wheel
Please read https://software.intel.com/en-us/articles/intel-optimization-for-tensorflow-installation-guide for more details about CPU options

```bash
./configure
```

This is the configure of this wheel:
```bash
./configure 
WARNING: Output base '/home/damian/.cache/bazel/_bazel_damian/9fcb7b127fd29ce682f779e62c1e54f6' is on NFS. This may lead to surprising failures and undetermined behavior.
WARNING: ignoring _JAVA_OPTIONS in environment.
WARNING: --batch mode is deprecated. Please instead explicitly shut down your Bazel server using the command "bazel shutdown".
You have bazel 0.24.0- (@non-git) installed.
Please specify the location of python. [Default is /home/damian/.local/easybuild/Debian9/software/Python/3.7.0-intel-2018b/bin/python]: 


Found possible Python library paths:
  /home/damian/.local/easybuild/Debian9/software/Python/3.7.0-intel-2018b/lib/python3.7/site-packages
  /hpc/eb/Debian9/EasyBuild/3.8.0/lib/python2.7/site-packages
Please input the desired Python library path to use.  Default is [/home/damian/.local/easybuild/Debian9/software/Python/3.7.0-intel-2018b/lib/python3.7/site-packages]

Do you wish to build TensorFlow with XLA JIT support? [Y/n]: 
XLA JIT support will be enabled for TensorFlow.

Do you wish to build TensorFlow with OpenCL SYCL support? [y/N]: 
No OpenCL SYCL support will be enabled for TensorFlow.

Do you wish to build TensorFlow with ROCm support? [y/N]: 
No ROCm support will be enabled for TensorFlow.

Do you wish to build TensorFlow with CUDA support? [y/N]: 
No CUDA support will be enabled for TensorFlow.

Do you wish to download a fresh release of clang? (Experimental) [y/N]: 
Clang will not be downloaded.

Do you wish to build TensorFlow with MPI support? [y/N]: 
No MPI support will be enabled for TensorFlow.

Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native -Wno-sign-compare]: 


Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]: 
Not configuring the WORKSPACE for Android builds.

Preconfigured Bazel build configs. You can use any of the below by adding "--config=<>" to your build command. See .bazelrc for more details.
	--config=mkl         	# Build with MKL support.
	--config=monolithic  	# Config for mostly static monolithic build.
	--config=gdr         	# Build with GDR support.
	--config=verbs       	# Build with libverbs support.
	--config=ngraph      	# Build with Intel nGraph support.
	--config=numa        	# Build with NUMA support.
	--config=dynamic_kernels	# (Experimental) Build kernels into separate shared objects.
Preconfigured Bazel build configs to DISABLE default on features:
	--config=noaws       	# Disable AWS S3 filesystem support.
	--config=nogcp       	# Disable GCP support.
	--config=nohdfs      	# Disable HDFS support.
	--config=noignite    	# Disable Apache Ignite support.
	--config=nokafka     	# Disable Apache Kafka support.
	--config=nonccl      	# Disable NVIDIA NCCL support.
Configuration finished
```
Build a wheel for the LISA/Cartesius normal CPU partitions (AVX2 and AVX512). You can build a debug/profiling wheel by adding ```--copt=-g```
```bash
bazel build --config=mkl --cxxopt=-D_GLIBCXX_USE_CXX11_ABI=0 -c opt --copt=-mavx2 --copt=-mfma --copt=-O3 --copt=-mavx512f --config=numa --config=noaws --config=nogcp --config=nohdfs --config=noignite --config=nokafka --config=nonccl //tensorflow/tools/pip_package:build_pip_package 
```

### Install the wheel in your current Python environment
Unless set otherwise, Bazel writes in ```~/.cache/bazel/```. In my case it was ```~/.cache/bazel/_bazel_damian/9fcb7b127fd29ce682f779e62c1e54f6```. Please insert your local path before running the package building commands
```bash
./bazel-bin/tensorflow/tools/pip_package/build_pip_package --nightly_flag ${HOME}/.cache/bazel/_bazel_damian/9fcb7b127fd29ce682f779e62c1e54f6
pip install ${HOME}/.cache/bazel/_bazel_damian/9fcb7b127fd29ce682f779e62c1e54f6/tf_nightly-2.0.0a0-cp37-cp37m-linux_x86_64.whl --user --force-reinstall
```
