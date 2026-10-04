---

title: "Building CLI-Managed ROCm Containers for Python Pipelines"
date: 2026-10-04
draft: false
------------
# Building CLI-Managed ROCm Containers for Python Pipelines

This guide shows how to build a small, script-driven Docker workflow for GPU-enabled Python pipelines.

The approach is intentionally simple:

```text
                    Host
                     │
                     │ docker run / docker exec
                     ▼
             ┌─────────────────┐
             │    Container    │
             │                 │
             │  Base image     │
             │       │         │
             │       ▼         │
             │   /opt/venv     │
             │       │         │
             │       ▼         │
             │    Papermill    │
             │       │         │
             │       ▼         │
             │    Notebook     │
             └─────────────────┘
                     ▲
                     │
              bind-mounted
               project files
```

The Docker image provides the runtime.

The shell scripts configure the environment.

The project remains on the host.

That means changing project dependencies doesn't require rebuilding the base image.

---

## 1. Directory structure

A typical workspace can look like this:

```text
my_workspace/
├── container_management/
│   ├── setup.sh
│   └── run_notebook.sh
│
├── python_helpers/
├── rust_maturin_pkg/
│
├── nlp_project/
│   ├── requirements.txt
│   └── pipeline.ipynb
│
└── another_project/
    ├── requirements.txt
    └── pipeline.ipynb
```

The management scripts are independent of the individual projects.

---

# 2. Creating the container

The `setup.sh` script takes at least two arguments:

```bash
./setup.sh <container_name> <project_path>
```

Additional directories can optionally be supplied.

For example:

```bash
./setup.sh \
    pipeline_nlp \
    ../nlp_project \
    ../python_helpers \
    ../rust_maturin_pkg
```

The first argument identifies the container.

The second identifies the project that should become `/workDir`.

Everything after that is treated as an additional package directory.

---

## 3. GPU access

The container is started using the ROCm devices:

```bash
--device=/dev/kfd \
--device=/dev/dri
```

The script also configures the container for the shared-memory and debugging requirements of the workload:

```bash
--ipc=host \
--shm-size 16G \
--group-add video \
--cap-add=SYS_PTRACE
```

The project itself is mounted into:

```text
/workDir
```

So files created by the pipeline remain on the host.

The container can therefore be replaced without losing the project or generated notebook output.

---

# 4. Finding the ROCm-enabled Python

One of the less obvious parts of the setup is finding the correct Python interpreter.

The script doesn't simply assume that:

```bash
/usr/bin/python3
```

contains PyTorch.

Instead, it searches available Python executables and checks whether they can import PyTorch:

```bash
PY_BIN=""

for py in $(which -a python python3 python3.10 python3.11 2>/dev/null); do
    if $py -c "import torch" 2>/dev/null; then
        PY_BIN="$py"
        break
    fi
done
```

If no suitable interpreter is found, setup stops.

This is important because the purpose of the environment is to build on top of the PyTorch installation supplied by the base image.

---

# 5. Creating the project environment

Once the correct interpreter is found, the script creates:

```text
/opt/venv
```

using:

```bash
$PY_BIN -m venv --system-site-packages /opt/venv
```

The important part is:

```text
--system-site-packages
```

The base image already contains the PyTorch installation we want.

The virtual environment can therefore see those packages while providing an isolated location for project-specific packages.

After creation:

```bash
source /opt/venv/bin/activate
```

The script then installs the tools needed by the workflow:

```bash
pip install --quiet --upgrade \
    pip \
    setuptools \
    wheel \
    maturin \
    papermill \
    ipykernel
```

---

# 6. Project-specific requirements

If the project contains:

```text
requirements.txt
```

the script automatically installs it:

```bash
if [ -f "/workDir/requirements.txt" ]; then
    pip install --quiet -r /workDir/requirements.txt
fi
```

This is one of the main advantages of the setup.

Changing:

```text
requirements.txt
```

doesn't require rebuilding the Docker image.

The project environment is simply prepared again when the container is created.

---

# 7. Registering the Python environment as a Jupyter kernel

The virtual environment is registered explicitly:

```bash
python -m ipykernel install \
    --sys-prefix \
    --name opt_venv \
    --display-name "Python (opt_venv)"
```

This gives the container a known kernel name:

```text
opt_venv
```

That kernel is later selected explicitly by Papermill.

This avoids relying on whichever Python environment happens to be the default Jupyter kernel.

---

# 8. Mounting additional Python packages

The setup script also accepts additional directories:

```bash
./setup.sh \
    pipeline_nlp \
    ../nlp_project \
    ../python_helpers \
    ../rust_maturin_pkg
```

These directories are mounted beneath:

```text
/extra_pkgs/
```

For example:

```text
/extra_pkgs/pkg_0
/extra_pkgs/pkg_1
```

Their locations are then added to `PYTHONPATH`.

The script persists that setting by adding it to:

```text
/opt/venv/bin/activate
```

This means that activating the environment automatically makes the mounted packages available.

This is particularly useful for shared helper modules or source packages that are still under active development.

---

# 9. Running notebooks with Papermill

Once a container has been created, notebooks can be executed using:

```bash
./run_notebook.sh \
    pipeline_nlp \
    pipeline.ipynb \
    results/nlp_output.ipynb
```

The script first checks that the requested container is actually running.

It then creates the output directory inside `/workDir` if necessary.

Finally, Papermill is executed inside the container:

```bash
source /opt/venv/bin/activate

papermill \
    '/workDir/pipeline.ipynb' \
    '/workDir/results/nlp_output.ipynb' \
    -k opt_venv
```

The important part is:

```bash
-k opt_venv
```

The notebook therefore runs using the kernel that was explicitly registered during setup.

---

# 10. Skipping tagged cells

The runner also supports an optional cell tag:

```bash
./run_notebook.sh \
    pipeline_nlp \
    pipeline.ipynb \
    results/output.ipynb \
    skip_cell
```

Papermill is then called with:

```bash
--skip-tagged-notebook-cells skip_cell
```

This can be useful when notebooks contain expensive setup, debugging, or interactive cells that shouldn't run during a particular pipeline execution.

---

# 11. Multiple pipelines, multiple containers

The container name is deliberately part of the interface.

For example:

```bash
./setup.sh pipeline_nlp ../nlp_project
./setup.sh pipeline_vision ../vision_project
```

These become two separate environments:

```text
pipeline_nlp
    └── nlp_project
        └── requirements.txt

pipeline_vision
    └── vision_project
        └── requirements.txt
```

Each pipeline can therefore have different dependencies without polluting the other environment.

This is especially useful when projects have incompatible Python packages or different hardware requirements.

---

# 12. Why not build a Docker image for every project?

You certainly can.

A traditional Docker workflow might look like:

```text
Dockerfile
    ↓
docker build
    ↓
project image
    ↓
docker run
```

The approach here deliberately moves some of that configuration to runtime:

```text
base image
    ↓
docker run
    ↓
runtime setup
    ↓
project environment
```

The trade-off is important.

### Building everything into an image gives you:

* immutable environments
* faster repeated startup
* stronger reproducibility

### Runtime setup gives you:

* faster iteration during development
* no image rebuild for every dependency change
* easier project-specific configuration
* a very small amount of Docker infrastructure

For my workflow, the second approach was the better fit.

It isn't universally better.

It's simply a better match for an environment where the heavyweight base image changes rarely while the project dependencies change frequently.

---

# 13. The next evolution: configurable base images

There is currently one important limitation.

The base image is hard-coded:

```bash
rocm/pytorch
```

That works for GPU-heavy pipelines.

It doesn't make much sense for a pipeline that doesn't need ROCm.

The next step is therefore to make the image another parameter of the setup script.

Conceptually:

```bash
./setup.sh \
    pipeline_nlp \
    ../nlp_project \
    rocm/pytorch
```

could start a ROCm environment, while:

```bash
./setup.sh \
    pipeline_cpu \
    ../cpu_project \
    <cpu-oriented-image>
```

could use a completely different runtime.

The architecture then becomes:

```text
                  setup.sh
                     │
          ┌──────────┴──────────┐
          │                     │
      project               base image
          │                     │
          ▼                     ▼
    requirements          hardware/runtime
          │                     │
          └──────────┬──────────┘
                     ▼
                 container
                     │
                     ▼
                  /opt/venv
                     │
                     ▼
                 pipeline
```

At that point, the scripts are no longer really "ROCm container scripts".

They become a small, generic **project container runner** with support for different runtime images.

ROCm is simply one of the available backends.

---

# 14. Things to keep in mind

There are a few important limitations to this approach.

### Containers are disposable

Anything not stored in a mounted directory or a persistent volume disappears when the container is removed.

That's intentional here.

### Runtime installation costs something

You save Docker build time, but the Python environment still has to be prepared.

This is a trade-off, not magic.

### Native Python packages need the right ABI

If a package contains compiled extensions, don't assume a wheel built on the host will work inside the container.

For example:

```text
host:       CPython 3.14
container:  CPython 3.12
```

A `cp314` wheel cannot simply be installed into CPython 3.12.

Build native packages for the Python environment that will actually execute them.

### The base image still matters

The virtual environment doesn't magically turn a CPU-only image into a ROCm environment.

The base image is still responsible for providing the underlying runtime.

That is precisely why making the base image configurable is the logical next step.

---

# 15. The resulting workflow

The final workflow is deliberately small:

```bash
# Start/configure a project container
./setup.sh pipeline_nlp ../nlp_project ../python_helpers

# Execute a notebook
./run_notebook.sh \
    pipeline_nlp \
    pipeline.ipynb \
    results/output.ipynb

# Stop when finished
docker rm -f pipeline_nlp
```

The important part isn't the commands themselves.

It's the separation of responsibilities:

```text
Docker image
    → runtime

setup.sh
    → environment

requirements.txt
    → project dependencies

bind mounts
    → source/shared code

run_notebook.sh
    → execution

container name
    → pipeline isolation
```

That separation is what made the whole setup manageable.

And the next logical step is to remove the final hard-coded assumption: **the runtime image should be chosen by the project rather than by the script.**

