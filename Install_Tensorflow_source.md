This gives the steps to build Tensorflow 2.0 for the CPU partitions on LISA/Cartesius. It can be reused for other systems.

If all steps are followed in sequence, you should be able to obtain a working Tensoflow installation. If needed, remember to add the ```-g``` flag to the ```bazel build``` step. This is tested for Python 3.7, however it should work the same for any Python 2.7 stack.

# Load pre-requisites
```bash 
module load GCCcore/7.3.0
module load Java/1.8.0_152
module load Python/3.7.0-intel-2018b
module load hwloc/1.11.10-GCCcore-7.3.0
```

If you are missing the 2018b suite, you can use any other GCC 5+ toolchain. On LISA/Cartesius you can install it with:
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

### Build the TF2.0 Python wheel
Please read https://software.intel.com/en-us/articles/intel-optimization-for-tensorflow-installation-guide for more details about CPU options

```bash
bazel build --config=mkl --cxxopt=-D_GLIBCXX_USE_CXX11_ABI=0 -c opt --copt=-mavx --copt=-mavx2 --copt=-mfma --copt=-O3 --copt=-mavx512f --copt=-mavx512pf --copt=-mavx512cd --copt=-mavx512er //tensorflow/tools/pip_package:build_pip_package
```

### Install the wheel in your current Python environment
```bash
./bazel-bin/tensorflow/tools/pip_package/build_pip_package --nightly_flag /tmp/tensorflow_pkg
pip install /tmp/tensorflow_pkg/tensorflow-2-0.whl
```
