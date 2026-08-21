---
title: "Rebuilding the Rust/Python Bridge: Starting With a Proof of Concept"
date: 2026-08-20
draft: false
tags:
  - rust
  - python
  - pyo3
  - maturin
  - machine-learning
  - jupyter
description: "Restarting the Rust-to-Python wrapper with a minimal PyO3 and Maturin proof of concept before rebuilding the real integration."
featured: true
---

# Rebuilding the Rust/Python Bridge: Starting With a Proof of Concept

I already have the application.

I already have the API.

What I don't currently have is a working Python wrapper around the Rust implementation.

The original wrapper was written against an older version of PyO3 and an older Maturin workflow. The ecosystem has moved on, and some of the code no longer fits the current APIs.

Rather than spending a lot of time trying to resurrect the old wrapper line by line, I'm taking a step back.

The first goal is simple:

> Get a Rust function callable from Python again.

That's it.

Once that works, I have a known-good foundation from which I can rebuild the actual wrapper.

This post is therefore less about building something useful and more about **getting the integration layer back on solid ground**.

## The bigger picture

The application I'm working toward is an ML application with a Rust implementation. Eventually, I want the Rust code to be usable from Python and, more importantly, usable naturally inside Jupyter-based machine-learning workflows.

The architecture I am aiming for looks roughly like this:

```text
┌───────────────────────────────┐
│        Python / Jupyter       │
│                               │
│        Python package         │
└───────────────┬───────────────┘
                │
                │ PyO3
                ▼
┌───────────────────────────────┐
│        Rust wrapper            │
│                               │
│      Python-facing API        │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│       Rust ML algorithm       │
│                               │
│       actual implementation   │
└───────────────────────────────┘
```

There are actually two related pieces of work here, and I want to keep them separate.

This series is about the **Rust/Python wrapper**: PyO3, Maturin, packaging, Python APIs, data exchange, and eventually Jupyter integration.

The Rust implementation of the ML algorithm will be covered in a **separate series**.

That separation matters. The algorithm itself is one engineering problem. Making that algorithm usable from Python is another one.

For now, I am concentrating on the second problem.

## Why start over?

The old wrapper was written against an older PyO3 API.

For example, the module definition used to look like this:

```rust
use pyo3::prelude::*;

/// A simple function exposed to Python
#[pyfunction]
fn double(x: usize) -> usize {
    x * 2
}

/// Define the Python module
#[pymodule]
fn rust_python_wrapper(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(double, m)?)?;
    Ok(())
}
```

The newer API changed the module signature. The equivalent code now involves `Bound`:

```rust
use pyo3::prelude::*;

/// A simple function exposed to Python
#[pyfunction]
fn double(x: usize) -> usize {
    x * 2
}

/// Define the Python module
#[pymodule]
fn rust_python_wrapper(m: &Bound<'_, PyModule>) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(double, m)?)?;
    Ok(())
}
```

This is not necessarily a huge change in terms of what the program does, but it is enough to make an older wrapper require some work.

The `Bound` API also introduces concepts around lifetimes and ownership that I don't want to debug at the same time as I am trying to recover the overall build and packaging workflow.

There is also a newer, considerably simpler module syntax available:

```rust
use pyo3::prelude::*;

#[pyfunction]
fn calc(x: usize) -> usize {
    x * 2
}

#[pymodule]
mod rust_python_wrapper {
    #[pymodule_export]
    use super::calc;
}
```

For the proof of concept, this is the approach I am using.

The important thing isn't which syntax is ultimately going to be used by the finished package.

The important thing is:

**Does the current toolchain work?**

## The smallest possible experiment

I created a new Rust library crate:

```bash
cargo new --lib python_rust_wrapper_hello
```

This gives me a completely clean environment instead of another attempt to fix the existing project while simultaneously changing its architecture.

The Rust implementation is intentionally ridiculous:

```rust
use pyo3::prelude::*;

#[pyfunction]
fn calc(x: usize) -> usize {
    x * 2
}

#[pymodule]
mod python_rust_wrapper_hello {
    #[pymodule_export]
    use super::calc;
}
```

The function does exactly one thing:

```text
5 → 10
```

That's enough.

If Python can call this function, the fundamental Rust-to-Python path is working.

## Cargo.toml

The project only needs PyO3 as a dependency:

```toml
[package]
name = "python_rust_wrapper_hello"
version = "0.1.0"
edition = "2024"

[dependencies]
pyo3 = { version = "0.29" }
```

I'm deliberately keeping the dependency list as small as possible.

This isn't the final package configuration. It is a test.

## The Python side

The Python application is equally boring:

```python
import python_rust_wrapper_hello

print(python_rust_wrapper_hello.calc(5))
```

There is no ML code.

There is no NumPy.

There is no Jupyter integration.

There isn't even an interesting algorithm.

That's the point.

I want to test one boundary at a time.

## Setting up the Python environment

I created a virtual environment directly inside the project:

```bash
python -m venv .venv
```

Then activated it:

```bash
source .venv/bin/activate
```

And installed Maturin:

```bash
pip install maturin
```

The virtual environment keeps this experiment isolated from the system Python installation.

That is particularly useful when working with native Python extensions, because the Python version and platform are part of the environment in which the extension is being built and tested.

## Building the extension

With the environment active:

```bash
maturin build --release
```

Maturin builds the Rust library and creates a Python wheel.

The resulting wheel is placed under:

```text
target/wheels/
```

This is an important step in the experiment.

I'm not just compiling a Rust library and manually loading it from Python. I'm testing the actual Python packaging path.

The goal is eventually to have something that can be installed like a normal Python package.

## Installing the wheel

I installed the generated wheel with:

```bash
pip install target/wheels/*
```

Now the Python environment has the compiled Rust extension installed.

## Running the test

Finally:

```bash
python test_app.py
```

The result:

```text
10
```

That's it.

The proof of concept works.

Python called a function implemented in Rust through PyO3, and the resulting extension was built and installed as a Python wheel using Maturin.

## What I have actually proven

Not very much.

And that's exactly what I wanted.

The experiment establishes that the basic toolchain works:

```text
Rust
  ↓
PyO3
  ↓
Maturin
  ↓
Python wheel
  ↓
Python
```

I now have a current, working reference implementation against which I can compare the real wrapper.

If the existing application fails to build after this point, I know the problem is in the application or wrapper rather than in the basic Rust/Python setup.

That distinction is useful.

## What I haven't solved yet

Almost everything interesting.

A real ML wrapper introduces a completely different set of problems.

For example, I will eventually need to figure out how to move data between Python and Rust efficiently.

Passing a single integer is easy:

```python
calc(5)
```

Passing a large NumPy array is a different problem.

If the wrapper copies every dataset between Python and Rust, I could easily end up throwing away much of the performance benefit of using Rust in the first place.

That means questions about:

- NumPy interoperability
- memory ownership
- borrowing
- lifetimes
- zero-copy access
- numeric types
- error handling
- Python exceptions
- Rust data structures
- API design
- testing
- packaging
- platform-specific wheels

will become important fairly quickly.

But those are problems for the next steps.

I don't need to solve them to prove that:

```text
Rust → Python
```

works.

## Why not fix the old wrapper directly?

This is probably the most important engineering decision in this first step.

I could take the existing wrapper and start changing code until it compiles again.

That would probably work eventually.

But then I would have several variables changing simultaneously:

- the PyO3 API
- the Maturin configuration
- the wrapper implementation
- the existing application
- potentially the Python package structure

If something breaks, I would have to figure out which change caused it.

By starting with a tiny project, I have a known-good baseline.

The experiment is cheap.

The real wrapper isn't.

So I'm spending a little time establishing the baseline before touching the larger system.

## This is not the finished package

The `calc()` function isn't the product.

It isn't even particularly useful.

It's a screwdriver used to check whether the toolbox works.

The actual goal is to get back to the existing application and start exposing meaningful functionality through Python.

The progression should look something like:

```text
Current PyO3/Maturin POC
          ↓
First real wrapper function
          ↓
Real application data
          ↓
NumPy interoperability
          ↓
Python package
          ↓
Testing and packaging
          ↓
Jupyter integration
          ↓
ML pipeline
```

At some point, the Python side should look more like an actual library:

```python
from my_ml_package import Model

model = Model(...)
model.fit(X_train, y_train)

predictions = model.predict(X_test)
```

The Python user shouldn't need to care that the implementation underneath is Rust.

That's the eventual goal of the wrapper.

## Two series, one project

I'm splitting the work into two blog series because I think the two problems deserve their own space.

### Series 1: The Rust/Python wrapper

This series will follow the integration work:

- rebuilding the wrapper against current PyO3 APIs
- Maturin and Python packaging
- designing the Python-facing API
- handling Python and Rust data
- NumPy integration
- testing
- wheels and distribution
- Jupyter integration
- eventually using the package in an ML pipeline

### Series 2: The Rust ML algorithm

The second series will cover the actual Rust implementation:

- the algorithm itself
- data structures
- implementation decisions
- performance
- correctness
- testing
- potentially benchmarking and optimization

Eventually the two series meet.

The algorithm provides the functionality.

The wrapper makes that functionality usable from Python.

## What's next?

Now that the hello-world wrapper works, the next step is to stop playing with `calc()` and start rebuilding the actual integration.

The interesting work begins when the first real piece of the existing API crosses the Python/Rust boundary.

That's where I expect the problems to get considerably more interesting.

And that's also where this stops being a PyO3 hello-world exercise and starts becoming the project I actually want to build.

For now, though:

```text
5 → Rust → Python → 10
```

Good enough.

The bridge is working again.
