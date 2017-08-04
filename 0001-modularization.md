# ROOT Modularization

* Proposal: [ROOT-0001](0001-modularization.md)
* Authors: [Oksana Shadura](https://github.com/oshadura)
* Review Manager: 
* Status:
* Decision Notes:
* Bug:

### Introduction
Represented in ROOT Planning Meeting 2017 as a set of separate deliverables:
1. Essential for opening ROOT to collaborators willing to contribute with a large chunk of functionality 
2. Develop a model for building, distributing and deploying ROOT modules that can extend an existing installation of ROOT on demand
3. Evolve ROOT into `BOOT` (à la R) 
4. Package the current ROOT TFile plugins as ROOT modules 
5. Cope with package/module dependencies by interfacing ROOT with popular package managers (e.g. Spack, Conda, …)”

### Motivation

ROOT is missing a possibility of lightweight installation, that could allow in a comfortable way setup up minimal environment for a user. In ROOT, all sources in master are ~700 Mb, without any possibility to separate core libraries and secondary or specific libraries. 
The second reason is that a graph of library dependencies for ROOT software package are very complex and could be simplified.

### Proposed solution

Main idea is to provide modularisation of ROOT and easy separability in multiple discrete and independent modules. Packaging system with help to get rid of complex library dependencies that are introduced in current product status.

### Ideas

One of possible proposals is to introduce a "TInstaller" which could manage access to dependencies, subpackages and be able to install missing parts. 

One of ideas is to provide a “homebrew” like tool that will be able to provide cross platform installation of modules (libraries), with dependencies based on recipes. 
The main idea is to try to have a minimal installation of ROOT (libCore, libCling, libRio..) and running session from prompt, while calling a missing in installation ROOT object or function, will be proposed possibility to install missing component and from the current session (without changing it) to use it.

“Homebrew” packaging system by itself is Git-based system with allow user or developer easily upload, synchronize, and extend existing modules - recipes. Homebrew is supported on Mac OS X, has a fork for Linux - LinuxBrew, and there are some efforts in porting it to Windows (need to be tested). Homebrew by itself already has a recipe of ROOT version 5.34.36 and 6.08.10, so first step could be to update existing recipe for ROOT6 and other tags. The second alternative is to develop "ROOTBrew" system. 

### Detailed design

### Source compatibility

### Alternatives considered



