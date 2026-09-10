---

title: "Browser-side Growing Neural Gas"
date: 2026-09-10
tags: ["rust", "wasm", "machine-learning", "Linux", "web dev"]
description: "Interactive browser-based Growing Neural Gas application built with Rust and WebAssembly"
-------------------------------------------------------------------------------------------------------

# Browser-side Growing Neural Gas

An experimental implementation of **Growing Neural Gas (GNG)** running entirely in the browser.

The project is written in **Rust** and compiled to **WebAssembly (Wasm)**, allowing the neural gas to be trained directly in the browser without requiring server-side computation.

**Project status:** This is an evolving MVP/alpha. The demo and its interface are still being developed, and the version shown here will change as the project progresses.

---

## Interactive demo

The demo is organized around three steps: **Dataset**, **Parameters**, and **Run**.

### 1. Choose a dataset

Start in the **Dataset** tab and select the dataset you want the Growing Neural Gas to learn.

The selected data is used as the input for the training process.

### 2. Adjust the parameters

The **Parameters** tab contains the configuration of the neural gas.

The default values provide a starting point, but you can experiment with them to see how they affect the resulting network. Depending on the dataset, different parameter settings can lead to noticeably different structures and convergence behavior.

Changing the parameters is optional — you can simply use the defaults if you want to see the algorithm in action.

### 3. Run the training

When you're ready, switch to the **Run** tab.

There are currently two ways to train the network:

* **Run Net** — runs the complete training process automatically.
* **Run Steps** — advances the training one step at a time, allowing you to observe how the neural gas develops incrementally.

The step-wise mode is particularly useful for exploring how nodes and connections evolve during training.

### Resetting the network

**Init GNG** resets the Growing Neural Gas and creates a fresh network.

This is useful when experimenting with different parameters or when you simply want to start the training process again from scratch.

## What's under the hood?

The core algorithm is a **Growing Neural Gas**, an unsupervised learning algorithm that incrementally builds a graph representation of a dataset.

Instead of fixing the network structure beforehand, the neural gas can add and adapt nodes as training progresses. This makes the evolution of the graph itself part of the learning process.

The implementation is written in **Rust** and compiled to **WebAssembly**, so the computational part of the algorithm runs directly in the browser.

The goal of this project is also to explore the practical side of bringing a Rust-based machine-learning algorithm into a browser environment — from the algorithm itself through to its WebAssembly deployment and interactive visualization.

---

## An evolving project

This page intentionally presents the project in its current state rather than as a finished product.

The demo is being developed incrementally, and **the latest version is always deployed here**. The interface, available datasets, parameters, and visualization will therefore evolve over time.

The source repositories are not public yet, but they will be made available shortly.

For now, the interactive demo is the best way to explore the current state of the project.

---

## This demo runs completely in your browser

The application is currently an **MVP/alpha** and contains known bugs and missing functionality.


{{< wasm >}}

The clustering algorithm is implemented in **Rust** and compiled to **WebAssembly**.

Currently known issues include:

* **Run Net** does not train the model with the circles dataset.
* The screen is not cleared when changing datasets.
* Nodes from the previous neural gas can remain visible after changing datasets.
* The parameters can't be changed.

These issues are known and will be addressed as development continues.

Given the scope of the project and the other projects involved, I decided that releasing an early, working version is better than waiting indefinitely for a finished product.

This is therefore very much a **work in progress**. The version embedded above represents the current state of development and will be updated as the project evolves.
