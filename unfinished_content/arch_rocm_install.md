---
title: "Setting Up an AMD RX7900 GPU for Machine Learning on Arch Linux"
date: 2025-11-11
tags: ["machine learning", "amd", "rocm", "tensorflow", "docker", "arch linux"]
description: "Step-by-step guide to configuring an AMD RX7900 GPU with ROCm and running TensorFlow inside Docker on Arch Linux."
---

This guide walks through setting up an AMD RX7900 GPU on an Arch Linux system for machine learning workloads.  
TensorFlow will be running inside a Docker container configured to access the GPU via ROCm.

---

## 1. Install ROCm Driver

First, check if your GPU is detected:

```bash
lspci -k | grep -EA3 'VGA|3D'
```
Install the ROCm OpenCL runtime.
Before doing this, make sure your system is clean — remove any remnants of older GPU drivers to avoid conflicts.

```bash
sudo apt update
yay -S rocm-opencl-runtime
```
## 2. Install Docker
Install Docker from the Arch repositories.
The usermod command allows you to run Docker commands without needing sudo each time.

```bash
sudo pacman -S docker
sudo usermod -aG docker $USER
```

Then log out and back in, or run:
```bash
docker run -it --rm hello-world
```
## 3. Run the Test Container
We are using  the official Rocm jjcontainer:

```bash
docker run -it \
--network=host \
--device=/dev/kfd \
--device=/dev/dri \
  --group-add video \
  --cap-add=SYS_PTRACE \
  --security-opt seccomp=unconfined \
  rocm/tensorflow:latest
```
## 4. Check TensorFlow GPU Access
Once inside the container, open a Python shell and verify TensorFlow detects your GPU:

```python
python
```
Then run:

```python
import tensorflow as tf

print("Num GPUs Available: ", len(tf.config.list_physical_devices('GPU')))

gpus = tf.config.list_physical_devices('GPU')
print("GPUs:", gpus)
```
If everything is configured correctly, you should see your AMD GPU listed.

## 5. Run a TensorFlow Test
To make sure TensorFlow is actually using the GPU, try a quick computation:

```python

import tensorflow as tf

vec_1 = tf.constant([1.0, 2.0, 3.0], dtype=tf.float32)
vec_2 = tf.constant([2.0, 0.4, 30.0], dtype=tf.float32)
res = tf.multiply(vec_1, vec_2)
print("result:", res.numpy())
```

If the setup is correct, you’ll see the output tensor printed, confirming TensorFlow is operational.

Notes
If TensorFlow doesn’t detect your GPU, ensure your container has ROCm properly mapped and the correct runtime versions installed.

For troubleshooting, you can check the ROCm version inside the container with:

```bash
rocminfo
```
