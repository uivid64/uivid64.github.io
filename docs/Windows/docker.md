# docker container

# WSL 2 GPU Support for Docker Desktop on NVIDIA GPUs

It’s been a year since [Ben wrote about Nvidia support](https://www.docker.com/blog/wsl-2-gpu-support-is-here/) on Docker Desktop. At that time, it was necessary to take part in the Windows Insider program, use Beta CUDA drivers, and use a Docker Desktop tech preview build. Today, everything has changed:

- On the OS side, [Windows 11](https://www.microsoft.com/en-us/windows/windows-11) users can now enable their GPU without participating in the Windows Insider program. Windows 10 users still need to register.
- [Nvidia CUDA drivers](https://developer.nvidia.com/cuda/wsl) have been released.
- Last, the GPU support has been merged in Docker Desktop (in fact since version 3.1).

Nvidia used the term near-native to describe the performance to be expected.


### Where to find the Docker images

Base Docker images are hosted at https://hub.docker.com/r/nvidia/cuda. The original project is located at https://gitlab.com/nvidia/container-images/cuda.

### What they contain

The `nvidia-smi`utility allows users to query information on the accessible devices.

```
$ docker run -it --gpus=all --rm nvidia/cuda:11.4.2-base-ubuntu20.04 nvidia-smi
Tue Dec  7 13:25:19 2021
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.00       Driver Version: 510.06       CUDA Version: 11.6     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  On   | 00000000:01:00.0 Off |                  N/A |
| N/A    0C    P0    13W /  N/A |    132MiB /  4096MiB |     N/A      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------
```

The `dmon` function of nvidia-smi allows monitoring the GPU parameters :

```
$ docker exec -ti $(docker ps -ql) bash
root@7d3f4cbdeabb:/src# nvidia-smi dmon
# gpu   pwr gtemp mtemp    sm   mem   enc   dec  mclk  pclk
# Idx     W     C     C     %     %     %     %   MHz   MHz
    0    29    69     -     -     -     0     0  4996  1845
    0    30    69     -     -     -     0     0  4995  1844
```

The nbody utility is a CUDA sample that provides a benchmarking mode.

```
$ docker run -it --gpus=all --rm nvcr.io/nvidia/k8s/cuda-sample:nbody nbody -benchmark
...
> 1 Devices used for simulation
GPU Device 0: "Turing" with compute capability 7.5

> Compute 7.5 CUDA device: [NVIDIA GeForce GTX 1650 Ti]
16384 bodies, total time for 10 iterations: 25.958 ms
= 103.410 billion interactions per second
= 2068.205 single-precision GFLOP/s at 20 flops per interaction
```

Quick comparison to a CPU suggest a different order of magnitude of performance. GPU is 2000 times faster:

```
> Simulation with CPU
4096 bodies, total time for 10 iterations: 3221.642 ms
= 0.052 billion interactions per second
= 1.042 single-precision GFLOP/s at 20 flops per interaction
```

### What can you do with a paravirtualized GPU?

#### Run cryptographic tools

Using a GPU is of course useful when operations can be heavily parallelized. That’s the case for hash analysis. `dizcza` hosted its [nvidia-docker based images of hashcat](https://hub.docker.com/r/dizcza/docker-hashcat) on Docker hub. This image *magically* works on Docker Desktop!

```
$ docker run -it --gpus=all --rm dizcza/docker-hashcat //bin/bash
root@a6752716788d:~# hashcat -I
hashcat (v6.2.3) starting in backend information mode

clGetPlatformIDs(): CL_PLATFORM_NOT_FOUND_KHR

CUDA Info:
==========

CUDA.Version.: 11.6

Backend Device ID #1
  Name...........: NVIDIA GeForce GTX 1650 Ti
  Processor(s)...: 16
  Clock..........: 1485
  Memory.Total...: 4095 MB
  Memory.Free....: 3325 MB
  PCI.Addr.BDFe..: 0000:01:00.0
```

From there it is possible to run hashcat benchmark

```
hashcat -b
...
Hashmode: 0 - MD5
Speed.#1.........: 11800.8 MH/s (90.34ms) @ Accel:64 Loops:1024 Thr:1024 Vec:1
Hashmode: 100 - SHA1
Speed.#1.........:  4021.7 MH/s (66.13ms) @ Accel:32 Loops:512 Thr:1024 Vec:1
Hashmode: 1400 - SHA2-256
Speed.#1.........:  1710.1 MH/s (77.89ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
...
```

#### Draw fractals

The project at https://github.com/jameswmccarty/CUDA-Fractal-Flames uses CUDA for generating fractals. There are two steps to build and run on Linux. Let’s see if we can have it running on Docker Desktop. A simple Dockerfile with nothing fancy helps for that.

```
# syntax = docker/dockerfile:1.3-labs
FROM nvidia/cuda:11.4.2-base-ubuntu20.04
RUN apt -y update
RUN DEBIAN_FRONTEND=noninteractive apt -yq install git nano libtiff-dev cuda-toolkit-11-4
RUN git clone --depth 1 https://github.com/jameswmccarty/CUDA-Fractal-Flames /src
WORKDIR /src
RUN sed 's/4736/1024/' -i fractal_cuda.cu # Make the generated image smaller
RUN make
```

And then we can build and run:

```
$ docker build . -t cudafractal
$ docker run --gpus=all -ti --rm -v ${PWD}:/tmp/ cudafractal ./fractal -n 15 -c test.coeff -m -15 -M 15 -l -15 -L 15
```

Note that the `--gpus=all`is only available to the `run` command. It’s not possible to add GPU intensive steps during the `build`.

#### Machine learning

Well really, looking at GPU usage without looking at machine learning would be a miss. The `tensorflow:latest-gpu`image can take advantage of the GPU in Docker Desktop. I will simply point you to Anca’s blog earlier this year. She described a tensorflow example and deployed it in the cloud: https://www.docker.com/blog/deploy-gpu-accelerated-applications-on-amazon-ecs-with-docker-compose/

### Conclusion: What are the benefits for developers?

At Docker, we want to provide a turn key solution for developers to execute their workflows seamlessly:

- With Docker Desktop, developers can run their code locally and deploy to the infrastructure of their choice.
- We provide support in the issue tracker https://github.com/docker/for-win
- See what’s coming up and recommend feature requests in the Docker public roadmap https://github.com/docker/roadmap
- [Download](https://www.docker.com/products/docker-desktop) the latest version of Docker Desktop now.