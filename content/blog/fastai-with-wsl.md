---
title: "Setup fastai in WSL2"
tags: [WSL, Tutorial]
date: 2022-09-21
draft: false
---

Steps to get PyTorch and fastai working with CUDA support in WSL2.

<!--more-->

Lately, I have been working on [the deep learning course from fast.ai](https://course.fast.ai/). In the course, it's recommended to use cloud-hosted Jupyter notebooks, such as Kaggle and Colab. They are fine and the easiest way to get started, but I do miss my editor with all my favorite extensions that won't be available in a browser-based editor. At first, I wanted to set up it on my M1 laptop, but it seems like [the fastai library](https://docs.fast.ai/) [doesn't supports macOS yet](https://github.com/fastai/fastai/issues/3662#issuecomment-1168211736).

But then on last weekend, I suddenly remembered I do have a Windows PC sitting around! It's a Dell G5 (please don't judge) which I got during the pandemic lock down to play some League of Legends with friends, but I haven't been using it a lot since this year. It comes with an NVIDIA GeForce 1660 Super. Not the most powerful GPU, but should be better than using CPU. It also looks like I can just use WSL2 to create a Linux-based development environment while still keep my Windows around for video games. This really sounds like the best of both worlds!

After two days of mucking around, I finally got it working. It's mostly following the [NVIDIA guide](https://docs.nvidia.com/cuda/wsl-user-guide/index.html#getting-started-with-cuda-on-wsl). It is longer than I expected, and I definite forgot getting a development environment set up with Windows and Python can be quite tricky sometimes! But if there is only one thing I can suggest: make sure your environment is **using the same CUDA version everywhere**. Consistency turns out to be the key here!

# Install WSL

This part is easy, especially if you are on Windows 11 where WSL 2 no longer requires enrolling into the Insider program.

Microsoft has a good official [guide](https://learn.microsoft.com/en-us/windows/wsl/install). Make sure you use WSL 2 which is a lot nicer than WSL 1. Use `wsl --list --online` to list all available Linux distributions, and `wsl --install -d Ubuntu-20.04`. Do not use the 22.04 version from the Windows store because it has some issues with installing the CUDA Toolkit.

# Install NVIDIA driver

Use the [Advanced Driver Search](https://www.nvidia.com/Download/Find.aspx?lang=en-us) to search for the NVIDIA Driver. You want to find one that apparently should support your graphic card, and match the CUDA version that PyTorch/fastai depend on.

For example, as of now (2022/09/21), I am on `fastai==2.7.9` and `pytorch==1.12.1`. On [Conda](https://anaconda.org/pytorch-test/pytorch/files) you can find builds `py3.10_cuda11.6_cudnn8.3.2_0.tar.bz2`. So, when I picked the driver, I deliberately went for the one that is on CUDA 11.6 (for the record, I used `512.95`).

You would imagine that newer drivers (e.g. on CUDA 11.7) should also be compatible, but for some reason it's not. I was at first on the latest NVIDIA driver because I used it for gaming, so I kept it up-to-date. And I just couldn't get CUDA to work with PyTorch in WSL2. Eventually, I had to uninstall the latest driver and install the older version and the issue went away. 🤷‍♂️

# Install CUDA in WSL2

It's mostly following the [official guide](https://docs.nvidia.com/cuda/wsl-user-guide/index.html#cuda-support-for-wsl2), except for one catch: do not use the exact version in the instructions. For example, as of now the instructions use CUDA 11.7, but I should be using the older version that matches my drive that I install in the previous step.

You can find older CUDA Toolkit at the [CUDA Toolkit Archive](https://developer.nvidia.com/cuda-toolkit-archive). Make sure you select `WSL-Ubuntu` as the Distribution. I went for the [CUDA 11.6.1 version](https://developer.nvidia.com/cuda-11-6-1-download-archive?target_os=Linux&target_arch=x86_64&Distribution=WSL-Ubuntu&target_version=2.0&target_type=deb_local), and installed with the following instructions.

```bash
sudo apt-key del 7fa2af80
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/11.6.1/local_installers/cuda-repo-wsl-ubuntu-11-6-local_11.6.1-1_amd64.deb
sudo dpkg -i cuda-repo-wsl-ubuntu-11-6-local_11.6.1-1_amd64.deb
sudo apt-key add /var/cuda-repo-wsl-ubuntu-11-6-local/7fa2af80.pub
sudo apt-get update
sudo apt-get -y install cuda
```

# Install PyTorch and fastai.

I used [Miniconda](https://docs.conda.io/en/latest/miniconda.html) and [Mamba](https://mamba.readthedocs.io/en/latest/installation.html) to manage my Python environment. Once you have them installed, create a new environment for fastai.

```bash
conda create -n fastai
conda activate fastai
```

I installed PyTorch first to quickly test if CUDA is working correctly.

```bash
mamba install pytorch torchvision torchaudio cudatoolkit=11.6 -c pytorch -c conda-forge
```

Once PyTorch is installed, open a Python console and run the following.

```bash
% python
Python 3.10.6 | packaged by conda-forge | (main, Aug 22 2022, 20:35:26) [GCC 10.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import torch
>>> torch.cuda.is_available()
True
>>>
```

It looks like it's working! Then I proceeded to install fastai.

```bash
 mamba install -c fastchan fastai
```
