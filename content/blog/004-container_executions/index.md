

---
title: "Use containers directly instead of using VSCode"
date: 2026-10-04
tags:
    - Docker
    - GPU
    - ROCm
    - Linux
description: "From VS Code Dev Containers to CLI-Managed GPU Containers

"
featured: true
---


# From VS Code Dev Containers to CLI-Managed GPU Containers

*How I stopped rebuilding my development environment and started treating containers as disposable, configurable runtime environments.*

## It started with a simple idea

I wanted to use my AMD GPU for machine-learning workloads from inside VS Code.

The obvious solution seemed to be a Dev Container: put ROCm and PyTorch into a Docker image, let VS Code connect to it, and get a nice, isolated development environment.

In theory, this was exactly what I wanted.

In practice, I ran into two problems.

The first was the container lifecycle.

The container didn't always go away when I was finished with it. After leaving it running for a while, I would eventually notice the GPU fans spinning up. Something was still using the GPU, even though I wasn't actively doing anything.

The second problem was even more annoying.

Every small change to the Dockerfile could trigger a rebuild.

And rebuilding a ROCm-based image isn't exactly like rebuilding a tiny Alpine container.

A seemingly innocent change could turn a quick development iteration into a very long wait.

At some point I stopped asking myself:

> "How can I make Dev Containers work?"

and started asking:

> "Do I actually need Dev Containers?"

## Removing the IDE from the equation

The answer turned out to be no.

What I really needed was a way to:

1. start a GPU-enabled container,
2. mount my project into it,
3. prepare the Python environment,
4. run my pipeline,
5. stop the container when I'm done.

None of those things require VS Code.

So I moved the container lifecycle into shell scripts.

Instead of having VS Code manage the container, I could simply do something like:

```bash
./setup.sh pipeline_nlp ../nlp_project
```

and start exactly the environment I needed.

When I was finished, I could stop the container myself.

No IDE lifecycle to fight.

No remote-server warnings.

No mysterious container hanging around in the background.

Just Docker.

## The important change: stop rebuilding the image

The next realization was that I was putting too much into the Docker image.

The ROCm/PyTorch base image already contained the expensive part of the environment.

I didn't need to rebuild that image every time my project needed another Python package.

So I changed the architecture.

Instead of:

```text
Dockerfile
    ↓
large image build
    ↓
project environment
```

I moved toward:

```text
Base image
    +
Runtime setup
    +
Project requirements
    +
Mounted source code
```

The container itself becomes relatively disposable.

The environment is assembled when the container starts.

That makes iteration considerably faster because changing `requirements.txt` or adding another helper package doesn't require rebuilding the underlying ROCm image.

## The Python rabbit hole

This is where things became more interesting.

At first, it seemed obvious that I could simply create a virtual environment and install everything into it.

That turned out to be a bad assumption when ROCm PyTorch is involved.

The base image already contained a Python installation with the ROCm-enabled PyTorch I wanted.

Creating a completely isolated virtual environment and installing PyTorch again meant I could end up with a different PyTorch installation from the one provided by the image.

The solution was to create the virtual environment using the container's Python that already had PyTorch installed:

```bash
python -m venv --system-site-packages /opt/venv
```

The `--system-site-packages` part is important.

It allows the virtual environment to see the packages installed by the base image while still giving the project its own environment for additional packages.

The resulting separation is much more useful:

```text
ROCm/PyTorch base image
        │
        └── system Python + ROCm PyTorch
                    │
                    ▼
             /opt/venv
                    │
                    ├── project dependencies
                    ├── Papermill
                    ├── ipykernel
                    └── build tools
```

The script even searches for the Python interpreter that can actually import PyTorch rather than assuming a particular Python path.

That matters because container images aren't guaranteed to put their Python installation exactly where I expect it.

## From notebooks to pipelines

Once the container setup was under control, I also wanted to automate notebook execution.

Rather than treating Jupyter as an interactive development tool only, I started using notebooks as executable pipeline definitions.

That's where Papermill comes in.

The second script takes:

```bash
./run_notebook.sh \
    pipeline_nlp \
    train.ipynb \
    results/nlp_out.ipynb
```

and executes the notebook inside the running container.

The `/opt/venv` environment is registered as a Jupyter kernel:

```bash
python -m ipykernel install \
    --sys-prefix \
    --name opt_venv \
    --display-name "Python (opt_venv)"
```

Papermill can then explicitly use that kernel:

```bash
papermill ... -k opt_venv
```

This removes another source of ambiguity.

The notebook isn't just executed somewhere inside the container.

It is executed using the Python environment I prepared for that container.

## Projects get their own environments

The next problem was that I don't have just one pipeline.

Different projects have different dependencies.

So the setup script takes a project directory and optionally additional package directories:

```bash
./setup.sh \
    pipeline_nlp \
    ../nlp_project \
    ../shared_helpers \
    ../maturin_pkg
```

The project is mounted into:

```text
/workDir
```

and additional directories are mounted under:

```text
/extra_pkgs/
```

Their locations are added to `PYTHONPATH`.

This gives me a useful separation:

```text
Host
│
├── container_management/
│   ├── setup.sh
│   └── run_notebook.sh
│
├── shared_helpers/
├── maturin_pkg/
│
├── nlp_project/
│   └── requirements.txt
│
└── another_project/
    └── requirements.txt
```

Each project can now have its own container and dependency set.

For example:

```text
pipeline_nlp
    └── nlp_project/requirements.txt

pipeline_vision
    └── vision_project/requirements.txt
```

The containers don't need to share the same Python environment.

They only need to share the underlying runtime infrastructure.

## One container per pipeline

This is why the container name became an explicit argument.

```bash
./setup.sh pipeline_nlp ../nlp_project
```

and:

```bash
./setup.sh pipeline_vision ../vision_project
```

can create two independent environments.

That gives me something Dev Containers were originally supposed to provide: isolation.

But now the isolation is controlled explicitly by my scripts rather than by the IDE.

## The native-code problem

There was another lesson hiding in this setup.

Some of my own Python packages contain native code and are built using Maturin.

That introduced an important distinction between the host and the container.

For example, a wheel built on my host might target:

```text
cp314
```

while the container uses:

```text
cp312
```

Those are different Python ABIs.

A wheel built for CPython 3.14 isn't magically compatible with CPython 3.12.

That means native packages need to be built for the Python environment in which they will actually run.

This is another reason why building those packages inside the container can make more sense than simply mounting a host-built wheel.

The container isn't just a place where Python happens to run.

It defines the runtime that native extensions need to target.

## And now the next problem

At this point, the scripts are doing what I originally wanted.

But there is still an unnecessary assumption:

```bash
rocm/pytorch
```

is hard-coded into the setup script.

That's fine for my current pipelines.

It won't be fine forever.

I already have pipelines that don't need ROCm.

If a pipeline is CPU-only, there is little reason to start it from a heavyweight ROCm/PyTorch image.

So the next evolution is obvious:

**let the user choose the base image.**

Instead of:

```bash
./setup.sh pipeline_nlp ../nlp_project
```

I want something closer to:

```bash
./setup.sh pipeline_nlp ../nlp_project rocm/pytorch
```

or:

```bash
./setup.sh pipeline_cpu ../cpu_project python:3.12
```

The exact interface can still evolve, but the architectural idea is already clear.

The container manager shouldn't care whether the runtime is ROCm, CUDA, CPU-only Python, or something else.

It should receive an image and build the project environment on top of it.

## What I ended up building

What started as:

> "I want ROCm inside VS Code."

turned into something quite different.

I now have a small container orchestration layer that separates:

**Runtime**

The Docker base image provides the underlying system and hardware-specific software.

**Project**

The project directory is mounted into the container.

**Dependencies**

`requirements.txt` defines the Python dependencies for that project.

**Shared code**

Additional directories can be mounted and exposed through `PYTHONPATH`.

**Execution**

Papermill executes notebooks using a known Jupyter kernel.

**Lifecycle**

The shell scripts control when containers are created and destroyed.

And the next step is to make the runtime configurable by allowing each pipeline to choose its own base image.

## Was abandoning Dev Containers the right decision?

For my particular workflow, yes.

That doesn't mean Dev Containers are bad.

They solve a different problem very well: integrating containerized development directly into an IDE.

My problem was slightly different.

I wanted GPU-enabled containers that I could start, configure, execute, and destroy explicitly. I also wanted to avoid rebuilding a heavyweight image whenever my project dependencies changed.

For that workflow, a few shell scripts and `docker run` turned out to be a better abstraction than another layer of tooling.

The biggest lesson wasn't really about ROCm.

It was about separating **infrastructure from project state**.

Once the heavyweight runtime became a reusable base image, everything above it became much easier to change.
es kernel selection, cell parameterization, and output saving much cleaner than nbconvert.
