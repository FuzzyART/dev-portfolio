---
title: "Quick Sync A Linux backup tool"
date: 2026-09-28
tags: ["Rust", "Python", "Cybersecurity", "Linux", "product management"]
description: "Linux backup and restore tool built with Rust and Python, using rsync and JSON to provide safe, configurable file synchronization and backup management."
---

Quick Sync is a Linux backup tool that I am building for my own use while learning high stakes software development.

**Repository:** [FuzzyART/quick_sync](https://github.com/FuzzyART/quick_sync)

**Status: Active development**


# Quick Sync — Linux Backup & Restore Tool

The project started from a fairly simple problem: I wanted a backup solution that fit the way I actually use my Linux systems. I looked at existing solutions, but I didn't find one that quite matched what I was looking for. So, as is often the case with software projects, I decided to build my own.

The current implementation can synchronize selected files and directories using `rsync` and is configured through JSON. I am also developing a graphical interface for it, which will be merged into this project as development continues.

I am already using Quick Sync for my own backups.

That makes this project somewhat different from a typical portfolio project: **I actually depend on it.**

## Why build another backup tool?

There are plenty of backup solutions for Linux. This isn't an attempt to claim that the world needs yet another one.

I wanted something tailored to my own workflow, particularly around selecting what should be backed up, keeping the configuration explicit, and eventually making both backup and restoration manageable through a single application.

The long-term goal is to turn Quick Sync into a more complete backup management suite.

The planned functionality includes:

* Backup of selected files and directories
* Incremental backups
* Restore functionality
* Versioning for selected directories
* A graphical configuration interface
* Backup status and logging
* Safer validation of backup operations
* Eventually, a more complete management workflow around backup and recovery

The project is deliberately evolving. This page will evolve with it.

## Current state

At the moment, the project consists of a command-line backend and an independently developed GUI that I am in the process of merging.

The CLI currently works with JSON backup plans and provides operations for inspecting configurations, checking paths, performing dry runs and executing actual synchronization operations.

The repository also contains automated tests and CI work. The implementation is currently being restructured as the project grows.

**Current:**

* `rsync`-based file synchronization
* JSON configuration
* CLI interface
* Path validation
* Dry-run/check mode
* Automated tests
* GitLab CI integration
* Basic GUI under development

**In progress:**

* Merging the GUI with the main project
* Restore functionality
* More robust incremental backup behaviour
* Improved safety checks
* GUI-based configuration

**Planned:**

* Versioned backups
* More complete restore workflows
* Further safety and validation mechanisms
* A more comprehensive backup management interface

## The uncomfortable part: backups are dangerous

This project has also become a way for me to learn something that is difficult to learn with a typical tutorial project:

**software where getting something wrong can have serious consequences.**

With a web application, a typo might produce an error page.

With a backup application, a typo can potentially destroy data.

A missing `/`, an incorrect path, a wrong source/destination or a single incorrect character in a configuration can result in a synchronization operation doing something very different from what was intended.

If that operation runs against a NAS containing years of data, the consequences can be considerably worse than a failed unit test.

That is one of the reasons I am deliberately using this project as a learning exercise in developing software for higher-stakes situations.

I want to learn how to build software where **“it usually works” isn't good enough.**

That means thinking about things such as:

* Input validation
* Path validation
* Dry runs
* Failure handling
* Testing destructive operations
* Clear logging
* Explicit configuration
* Safe defaults
* Recoverability
* Understanding exactly what an operation will do before executing it

The project therefore isn't only about implementing a backup algorithm. It is also about learning how to make potentially destructive software behave predictably.

## Why `rsync`?

I don't see much value in reinventing the actual file synchronization mechanism.

`rsync` is already a mature and widely used tool for synchronizing files on Linux. Quick Sync therefore acts more as a management and orchestration layer around it.

The application is responsible for things such as configuration, validation, user interaction and eventually backup/restore management, while `rsync` handles the underlying synchronization.

This also gives me a useful engineering boundary between the application and the underlying system tool.

## What I am learning

This project is becoming one of my main exercises in practical software engineering.

I'm particularly interested in the difference between writing software that works on my machine and writing software that can be trusted with something important.

As the project grows, I expect the difficult problems to move increasingly away from simply getting files copied and toward questions such as:

> What exactly is going to happen?

> Can the user verify that before executing it?

> What happens when something fails halfway through?

> How do we know whether a backup is actually usable?

> Can the system recover from an interrupted operation?

> How do we prevent a perfectly valid command from being executed against the wrong location?

These are problems I want to learn by actually building and using the system.

## An evolving project

Quick Sync is not finished, and this page isn't intended to pretend that it is.

As the application develops, this project page will develop with it.

I want to document not only the features that eventually work, but also the design decisions, mistakes and problems encountered along the way.

The current version is already useful to me.

The goal is to gradually turn it into something much more complete: a Linux backup management suite that I can trust with my own data.


