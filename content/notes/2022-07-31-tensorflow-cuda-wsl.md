---
layout: post
title:  "Installing Tensorflow on WSL with CUDA support"
date:   2022-07-31 12:00:00 +0530
categories: workflow
tags:
    - tensorflow
    - cuda
    - wsl
    - gpu
---
[//]: # (The body)

Getting **`tensorflow`** to work properly on WSL with GPU support can be a little difficult. One needs to use the right dependencies for the a particular version of tensorflow.[^venv] The dependencies are listed [here][pipinstall] for the latest version. For older versions, it is listed [here instead][depends].

My hardware configuration looks like this:
```text
CPU: AMD Ryzen 5800H (8, 16)
GPU: NVIDIA GeForce RTX 3050Ti (Laptop)
RAM: 16 GB (8 GB allowed on WSL2)
```
Software:
```text
Ubuntu 22.04 LTS on WSL2
NVIDIA Graphics Driver (Game-ready) 516.59
CUDA toolkit 11.2 (11.7 supported)
cuDNN 8.1.1
Python 3.10.4
pip 22.2.1
tensorflow 2.9.1
```
Most of this is inspired from [this guide][wslguide1]. However, parts of it didn't work for me and I had to tweak.

We start with installing the cuda toolkit. In our case, we have to choose version 11.2 (the latest is 11.7), so we must dive into the NVIDIA [archives][cudatk-arxv] and choose the right version. Then we select **Linux -> x86_64 -> WSL-Ubuntu -> 2.0 -> deb (local)**. I use the local **deb** installer, however, this is upto the user.

At this point, if any other versions of cuda are installed, we can uninstall them with
```shell
sudo apt --purge remove cuda
sudo apt autoremove
```
This will work only if cuda was installed with apt. Otherwise, one can follow the instructions in this [stackoverflow thread][cudatk-uninst]. To remove references to the cuda repo (since the reference might be version-specific), we have two options:
  - Edit */etc/apt/sources.list* if it contains references to the NVIDIA cuda repo.
  - Delete the key in */etc/apt/sources.list.d* that refers to the same.
We also delete the old apt key:
```shell
sudo apt-key del 7fa2af80
```

Now we can proceed with the installation.[^drivernote] (The following is from the installation instructions for v11.2.0. The full guide for installing CUDA on WSL is [here][cuda-wsl]):
```shell
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/11.2.0/local_installers/cuda-repo-wsl-ubuntu-11-2-local_11.2.0-1_amd64.deb
sudo dpkg -i cuda-repo-wsl-ubuntu-11-2-local_11.2.0-1_amd64.deb
sudo apt-key add /var/cuda-repo-wsl-ubuntu-11-2-local/7fa2af80.pub
sudo apt-get update
sudo apt-get -y install cuda
```
Get a coffee - this might take time!

Meanwhile, we can edit our rc file (.bashrc or .zshrc or whatever is relevant) and add the following lines:
```shell
export PATH="/usr/local/cuda/bin:$PATH"
export LD_LIBRARY_PATH="/usr/local/cuda/lib64:$LD_LIBRARY_PATH"
```
Finally, we *source the rc file* to load the changes.

Now we install cuDNN. Installation instructions are [here][cudnn-inst]. We just get the library files of cuDNN and copy them to the appropriate location. We do not use an installer.
First, we install `zlib1g`, a compression library required by cuDNN (in my case, it was already installed):
```shell
sudo apt install zlib1g
```
Now we have to sign up for the NVIDIA Developer Program first. Once we're done and are signed in, we can proceed to downloading cuDNN. We go the [cuDNN archive][cudnn-arxv] and choose the appropriate version (v8.1.1 - the most recent version is listed [here instead][cudnn-down]). We choose the "cuDNN Library for Linux (x86_64)" option, which downloads a tar file. We move this tar file to our WSL distro (for quicker file ops) and extract this using
```shell
tar xvf <filename>
```
where *\<filename\>* should start with *cudnn-x.y-* where *x.y* is the cuda version.

From the folder in which files were extracted, we copy these files like so:
```shell
sudo cp <extracted-folder-name>/include/cudnn*.h /usr/local/cuda/include 
sudo cp -P <extracted-folder-name>/lib/libcudnn* /usr/local/cuda/lib64 
```
and change the permissions of these files to allow all users to read them:
```shell
sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
```
Finally, we install tensorflow:
```shell
pip install tensorflow
```
This will also, likely, take time. After that, we're done with the installation!

Now we can run this in a python shell to verify our 
```python
import tensorflow as tf
print(tf.config.list_physical_devices('GPU'))
```

If this shows a non-empty list, tensorflow recognizes the gpu and the necessary libraries.[^numawarn] Typically, this output will look something like

```python
[PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]
```










[//]: # (Footnotes, if any)

[^venv]: This naturally calls for the use of a virtual environment specifically for development involving `tensorflow` but we will be skipping this adventure here. This is traditionally done with **conda** - as most of the guides recommend. However, [issues][tf-conda] have been reported on WSL using this method. We can use a typical virtual environment (`virtualenv`) instead.
[^drivernote]: The display driver must be installed on **Windows**. No separate installation of a graphics drivers is required on WSL. Standard cuda toolkits for Linux include the driver - so we cannot choose those versions.
[^numawarn]: If a warning is given by tensorflow about `Your kernel may have been built without NUMA support.`: this is probably nothing to worry about, as discussed in [this thread][numaissue].





[//]: # (Links, if any)

[pipinstall]: <https://www.tensorflow.org/install/pip>
[depends]: <https://www.tensorflow.org/install/source#tested_build_configurations>
[wslguide1]: <https://medium.com/@xizengmao/install-tensorflow-with-gpu-acceleration-simultaneously-for-windows-and-wsl-linux-2-10da088d5e4f>
[cudatk-arxv]: <https://developer.nvidia.com/cuda-toolkit-archive>
[cudatk-uninst]: <https://stackoverflow.com/questions/56431461/how-to-remove-cuda-completely-from-ubuntu>
[cudnn-inst]: <https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html>
[cudnn-arxv]: <https://developer.nvidia.com/rdp/cudnn-archive>
[cudnn-down]: <https://developer.nvidia.com/rdp/cudnn-download>
[cuda-wsl]: <https://docs.nvidia.com/cuda/wsl-user-guide/index.html>
[numaissue]: <https://forums.developer.nvidia.com/t/numa-error-running-tensorflow-on-jetson-tx2/56119/4>
[tf-conda]: <https://stackoverflow.com/a/71058493/12983399>