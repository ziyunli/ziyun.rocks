+++
title = "Running ComfyUI on Mac"
date = "2024-10-26T11:02:15-07:00"
slug = "running-comfyui-on-mac"
+++

Setting up ComfyUI on a Mac requires a few specific steps, especially around managing dependencies and ensuring compatibility with Apple’s Metal acceleration. Here’s a detailed walkthrough of what worked for me.

## Prerequisites: Python Compatibility

ComfyUI requires Python 3.10 or lower; using a newer Python version can lead to errors, particularly when building the wheel for [Hugging Face’s `tokenizers`](https://github.com/huggingface/tokenizers/issues/1050). In my setup, I specified `python310` through [Nix](https://github.com/ziyunli/nix-home/blob/f973a101f951032db1df73f3e99ca1fb466abf93/home/packages.nix#L42) to avoid conflicts.

> **Note:** If you also have Homebrew installed, be aware that it may default to Python 3 for other packages. Homebrew's Python can override your setup, so it’s worth double-checking that you're running the correct Python version. Until I fully streamline my environment, I’m mindful of which Python is active and its version.

Verify your Python path and version as follows:

```bash
which python
# Output: ~/.nix-profile/bin/python

python --version
# Output: Python 3.10.15
```

## Step 1: Set Up a Virtual Environment

With the correct Python version confirmed, begin by cloning the ComfyUI repository and setting up a virtual environment to isolate dependencies.

```bash
git clone https://github.com/comfyanonymous/ComfyUI
cd ComfyUI
python -m venv venv
```

## Step 2: Enable Metal Acceleration

To optimize performance on Mac, we can use Metal acceleration, which accelerates PyTorch computations using Apple’s GPU framework. The feature is still in beta, so installing these packages requires some additional options.

1. **Activate the virtual environment:** Make sure you’re using the `venv` created in the previous step.
2. **Install PyTorch with Metal acceleration:** Run the following to install nightly versions of PyTorch with Metal support.

   ```bash
   ./venv/bin/pip install --pre torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/nightly/cpu
   ```

3. **Install ComfyUI dependencies:** Now, install the required packages for ComfyUI.

   ```bash
   ./venv/bin/pip install -r requirements.txt
   ```

## Step 3: Run ComfyUI

With dependencies installed, you’re ready to launch ComfyUI.

```bash
./venv/bin/python main.py
```

---

And that's it! You should now have ComfyUI running smoothly on your Mac with Metal acceleration. If any issues arise, revisit the Python version setup, as compatibility is key here.
