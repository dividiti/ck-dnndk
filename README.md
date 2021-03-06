# CK-DNNDK

A [Collective Knowledge](https://github.com/ctuning/ck) (CK) repository for
[DeePhi](http://www.deephi.com/)'s [DNNDK](http://www.deephi.com/dnndk.html)
with a portable workflow and reusable components.

[![logo](https://github.com/ctuning/ck-guide-images/blob/master/logo-powered-by-ck.png)](https://github.com/ctuning/ck)

# License

The files in this repository are licensed under [the CK license](LICENSE.CK)
(3-clause BSD), except the `main.cc` and `Makefile` files under `program/`
licensed under [the DNNDK license](LICENSE.DNNDK) (included with a written
permission from [Xilinx](http://xilinx.com) / [DeePhi Tech,
Inc.](http://deephi.com), as part of collaboration between
[Xilinx](http://xilinx.com) and [dividiti](http://dividiti.com)).

[![License](https://img.shields.io/badge/License-BSD%203--Clause-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)

# Setting up the host machine (Ubuntu workstation with CUDA)

## Installing CK

CK minimally requires:

* Python 2.7 or 3.3+ (the limitation is mainly due to unit tests).
* `git` (Git command line client).
* `wget`.

You can install CK in your local user space as follows:

```bash
user@host$ git clone http://github.com/ctuning/ck
user@host$ export PATH=$PWD/ck/bin:$PATH
user@host$ export PYTHONPATH=$PWD/ck:$PYTHONPATH
```
(You may want to put these into your `$HOME/.bashrc`.)

You can also install CK via `pip` to avoid setting up environment variables yourself either via the root account:
```
root@host# pip install ck
```
or via `sudo`:
```
user@host$ sudo pip install ck
```

Please see this [CK Installation](https://github.com/ctuning/ck#minimal-installation) guide for further details.
Please also feel free to read this [Getting Started](https://github.com/ctuning/ck/wiki/First-steps) guide to learn about basic CK concepts.


## Installing CK-DNNDK
Install the CK-DNNDK repository:
```bash
user@host$ ck pull repo:ck-dnndk
```

## Installing DeePhi DNNDK

### Installing dependencies
```bash
user@host$ sudo apt-get install -y --force-yes \
build-essential \
autoconf \
libtool \
libopenblas-dev \
libgflags-dev \
libgoogle-glog-dev \
libopencv-dev \
libprotobuf-dev \
protobuf-compiler \
libleveldb-dev \
liblmdb-dev \
libhdf5-dev \
libsnappy-dev \
libboost-system-dev \
libboost-thread-dev \
libboost-filesystem-dev \
libyaml-cpp-dev \
libssl-dev
```

**NB:** Many of these dependencies appear to be needed for Caffe, and are
probably statically linked into the DNNDK tools (so we shouldn't worry
about them). A few of the dependencies, however, are dynamically linked,
notably, Boost 1.58, HDF5 1.18, NCCL. The CK workflow takes care of them
as much as possible.

### Installing DNNDK
Download a DNNDK archive from [DeePhi](http://www.deephi.com/technology/dnndk),
save it to e.g. `~/Downloads/deephi_dnndk_v2.08_beta.tar.gz`, and run:

```bash
user@host$ ck install package --tags=lib,dnndk --extra_version=08 \
--env.CK_DNNDK_ARCHIVE_PATH=~/Downloads/deephi_dnndk_v2.08_beta.tar.gz
```

CK will detect your environment (Ubuntu 14.04 or 16.04; CUDA 8.0, 9.0, 9.1) and offer to install the compatible DNNDK package.

**NB:** The CK workflow can be easily extended when DNNDK supports other environments (e.g. Ubuntu 18.04; CUDA 10.0).

### Installing DNNDK models
To install models supported by DNNDK (when prompted for the installation path, press "Enter" to accept the default):
```bash
user@host$ ck install package:deephimodel-resnet50
user@host$ ck install package:deephimodel-inception-v1
user@host$ ck install package:deephimodel-vgg16
```

**NB:** You can also install the supported models interactively via:
```bash
user@host$ ck install package --tags=deephimodel

More than one package found:

 0) deephimodel-vgg16  Version fp32  (ea3b678fc516f374)
 1) deephimodel-resnet50  Version fp32  (23da52e1ee455394)
 2) deephimodel-inception-v1  Version fp32  (0ca4cfc443ba8794)
```

# Setting up the ZCU102 board

- Copy DNNDK headers, drivers, samples and images onto the board e.g.:
```bash
user@host$ ck virtual env --tags=lib,dnndk,v2
user@host$ scp -P 22 -r $CK_ENV_LIB_DNNDK/../ZCU102/* root@192.168.0.102:/root
```

- Log in to the board e.g.:
```bash
user@host$ ssh -p 22 root@192.168.0.102
```

**NB:** Depending on your board image, you may need to install `gcc`, `git` and other common utilities via `apt` before proceeding to the next step.

## Installing DNNDK
```bash
root@zcu102# cd /root && ./install.sh
Begin to install DeePhi DNNDK ...
Install DeePhi DPU Driver ...
Install DeePhi tools, runtime & libraries ...
Complete installation successfully.
The BOOT.BIN and system.dtb have been replaced.
The system will reboot after 5 seconds ...
5
4
3
2
1
user@host$
```

## Installing CK

- Log in to the board e.g.:
```bash
user@host$ ssh -p 22 root@192.168.0.102
```
- Install CK from GitHub:
```bash
root@zcu102# git clone https://github.com/ctuning/ck /root/CK
```
- Add the following lines to _both_ `/root/.bashrc` and `/root/.bash_profile`:
```bash
# Set up environment variables for Collective Knowledge.
export CK_ROOT=/root/CK
export PATH=$CK_ROOT/bin:$PATH
export CK_TOOLS=$HOME/CK_TOOLS
export CK_REPOS=$HOME/CK_REPOS
```
- Test CK:
```bash
root@zcu102# source /root/.bashrc
root@zcu102# ck version
V1.9.6.1
```

## Installing CK-DNNDK
```bash
root@zcu102# ck pull repo:ck-dnndk
```

## Installing the ImageNet validation set (50,000 images ~ 6.5 GB)
```bash
root@zcu102# ck install package:imagenet-2012-val
```
**NB:** For testing the workflows, you can also install a subset of images from this dataset (the first 500 images, ~64 MB):
```bash
root@zcu102# ck install package:imagenet-2012-val-min
```

### Resizing the dataset to `224x224`

As the ImageNet images come in all shapes in sizes, they need to be resized to the input of resolution of `224x224`. As resizing images at runtime (online) may affect performance, you can resize images ahead of time (offline) as follows:
```
root@zcu102# ck show env --tags=imagenet,val
Env UID:         Target OS: Bits: Name:                         Version: Tags:

5354ee8e40829858   linux-64    64 ImageNet dataset (validation) 2012     64bits,caffe,channel-stable,dataset,host-os-linux-64,ilsvrc2012,ilsvrc2012min,imagenet,raw,small_dataset,target-os-linux-64,v2012,val

root@zcu102# ck load env:5354ee8e40829858 --tags=imagenet,val | grep full_path
      "full_path": "/root/CK_TOOLS/dataset-imagenet-ilsvrc2012-val-min/ILSVRC2012_val_00000001.JPEG",

root@zcu102# find /root/CK_TOOLS/dataset-imagenet-ilsvrc2012-val-min -name *.JPEG \
-exec convert -resize 224x224! -interpolate bilinear {} \;
```

## Testing the models with prebuilt DPU binaries

You can test the models with 500 640x480 images
(copied to `/root/samples/common/image500_640_480/`)
using prebuilt DPU binaries provided in the DNNDK package as follows:

- ResNet50:
```bash
root@zcu102# ck compile program:image-classification-resnet50-deephi
root@zcu102# ck run program:image-classification-resnet50-deephi --cmd_key=default
...
Execution time: 19.050 sec.
```

- Inception-v1:
```bash
root@zcu102# ck compile program:image-classification-inception_v1-deephi
root@zcu102# ck run program:image-classification-inception_v1-deephi --cmd_key=default
...
Execution time: 13.725 sec.
```

# Converting DNNDK models

## Installing calibration images
```bash
user@host$ ck install package --tags=deephi,calibration
```
**FIXME:** This downloads the `ResNet50.tar.gz` package from DeePhi. This may avoided if one of the DeePhi model packages is already installed. (All of them contain these images, so they can simply be detected.)

## Converting models

To convert a Caffe model using DECENT, compile to ELF files using DNNC,
and copy the ELF files to the board in one go, run:
```bash
user@host$ ck run program:caffe2deephi --cmd_key=a-b-c:convert-compile-copy
```
You can customize any of the following runtime environment variables (the default values are specified in parentheses):

- `CK_BOARD_USER` (`root`);
- `CK_BOARD_PORT` (`22`);
- `CK_BOARD_ADDRESS` (`192.168.0.102`);
- `CK_CPU_ARCH` (`arm64`);
- `CK_DPU_TYPE` (`4096FA`);

by adding `--env.KEY=VALUE` to the command line.

For debugging, you can run the whole process step-by-step:
```bash
user@host$ ck run program:caffe2deephi --cmd_key=a:convert
user@host$ ck run program:caffe2deephi --cmd_key=b:compile
user@host$ ck run program:caffe2deephi --cmd_key=c:copy
```
The log files are located under the directory given with the command:
```bash
user@host$ ck find program:caffe2deephi
```
- `tmp/stdout.log`: standard output of the last executed command;
- `tmp/stderr.log`: standard error of the last executed command;
- `tmp/decent_out/decent.log`: log from the DECENT converter;
- `tmp/dnnc_out/dnnc.log`: log from the DNNC converted (contains a copy of a log from DECENT).

# Running DNNDK models

- Log in to the board e.g.:
```bash
user@host$ ssh -p 22 root@192.168.0.102
```

## Testing the models

- ResNet50:
```bash
root@zcu102# ck compile program:image-classification-resnet50-deephi --env.CK_DEEPHI_PROGRAM_ELF_DIR=../elf
root@zcu102# ck run program:image-classification-resnet50-deephi --cmd_key=default
```

- Inception-v1:
```bash
root@zcu102# ck compile program:image-classification-inception_v1-deephi --env.CK_DEEPHI_PROGRAM_ELF_DIR=../elf
root@zcu102# ck run program:image-classification-inception_v1-deephi --cmd_key=default
```

## Benchmarking the models

To measure accuracy on the ImageNet validation set (50,000 images), run:

- ResNet50:
```bash
root@zcu102# ck benchmark program:image-classification-resnet50-deephi \
--cmd_key=imagenet --repetitions=1 --env.CK_DEEPHI_PROGRAM_ELF_DIR=../elf \
--record --record_repo=local --record_uoa=dnndk-v2.08-resnet50 \
--tags=image-classification,deephi,zcu102,dnndk-v2.08,resnet50 \
--skip_print_timers --skip_stat_analysis --process_multi_keys
```

- Inception-v1:
```bash
root@zcu102# ck benchmark program:image-classification-inception_v1-deephi \
--cmd_key=imagenet --repetitions=1 --env.CK_DEEPHI_PROGRAM_ELF_DIR=../elf \
--record --record_repo=local --record_uoa=dnndk-v2.08-inception_v1 \
--tags=image-classification,deephi,zcu102,dnndk-v2.08,inception_v1 \
--skip_print_timers --skip_stat_analysis --process_multi_keys
```

## Archiving the experimental results on the board
```
root@zcu102# ck list local:experiment:*
dnndk-v2.08-inception_v1
dnndk-v2.08-resnet50
root@zcu102# ck zip local:experiment:* --archive_name=/root/ck-dnndk-xilinx-zcu102.zip
```
## Copying the experimental results to the host
```
user@host$ scp -P 22 root@192.168.0.102:/root/ck-dnndk-xilinx-zcu102.zip ~/Downloads
user@host$ ck add repo --zip=~/Downloads/ck-dnndk-xilinx-zcu102.zip
```
