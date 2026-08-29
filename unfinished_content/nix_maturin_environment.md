---
title: "Building a Python Package from a Rust Library with Maturin on Arch Linux"
date: 2025-11-11
tags: ["rust", "python", "maturin", "pyenv", "arch linux"]
description: "Step-by-step guide to building and installing a Python package from a Rust library using Maturin."
---

Setting up a Python package built from a Rust library can be surprisingly smooth with [Maturin](https://github.com/PyO3/maturin).  
Here’s the full process I used on Arch Linux.

---

## 1. Install and configure Python

I’m using **pyenv** to manage Python versions.  
Make sure you have both Python and pyenv installed.

```bash
sudo pacman -S python3.12
pyenv install 3.12.11
pyenv local 3.12.11

```

## 2. Create and activate a virtual environment

Inside your project directory:
```bash
python3.12 -m venv .venv
source .venv/bin/activate
```

## 3. Build the Rust project with Maturin

Once your virtual environment is active, build the package in release mode:
```bash
maturin build --release
```

This creates a .whl (wheel) file under target/wheels/.

## 4. Install the generated Python package

Replace the app_filename with the actual wheel generated in your case:
```bash
pip install app_filename/target/wheels/gng_py-0.1.1-cp312-cp312-linux_x86_64.whl
```

## 5. Set up your IDE

Open the project folder in VS Code, making sure .venv is in the project root.

Then, select the Python interpreter from the virtual environment:

Command Palette → Python: Select Interpreter → .venv

Done.