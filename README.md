# tensorflow2.0-support
Support documents and scripts for Tensorflow2.0

## Build & install Tensorflow
### LISA/Cartesius CPU
In its current nightly phase, it might be better to build the CPU-only wheel from sources. For that please follow the steps from https://github.com/sara-nl/tensorflow2.0-support/blob/master/Install_Tensorflow_source.md. 

#### With AVX512 support
The *debug* wheel can be downloaded directly from here https://surfdrive.surf.nl/files/index.php/s/xLEAkI7bf5kANc6/download.
The *production* wheel can be downloaded directly from here https://surfdrive.surf.nl/files/index.php/s/rf5I0RIM8ku15UA/download.

#### With AVX2 support


### LISA/Cartesius GPU
```bash
module load Python/3.6.3-foss-2017b
module load cuDNN/7.3.1-CUDA-10.0.130 
module load NCCL/2.3.5-CUDA-10.0.130
pip install tensorflow-gpu==2.0.0-alpha0 --user
```
