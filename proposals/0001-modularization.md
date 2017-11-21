# ROOT Modularization

* Proposal: [ROOT-0001](0001-modularization.md)
* Authors: [Brian Bockelman,  Oksana Shadura, Vassil Vassilev](https://github.com/oshadura)
* Review Manager: 
* Status:
* Decision Notes:
* Bug:

## Introduction

Since some decades now, there exist many legacy large object-oriented software systems  as ROOT consisting of a large number of interdependent  and loose-coupled classes, mostly organized as set of libraries and build targets.
In such systems, classes are at a low level of granularity to serve as a unit of software modularization. In object-oriented languages such as Java,  Python and C++, package structure allows people to organize their programs into components. A good organization of classes into identifiable and collaborating packages eases the understanding, maintenance of software. To improve the quality of software modularization, assessing the package organization is required.[1]
Since we address large software systems as ROOT framework, consisting of a very large number of classes and packages, we consider that packages are the units of software modularization. 
Are they, package and module are synonymous concepts? A module is a set of functions, types, classes and etc., defined in a common namespace. We can define module also as a set of related functionality that exposes a well-defined API. A library is a module or set of modules  which makes sense to be together and that can be used in a program or another library. A package is a unit of distribution that can contain a library or an executable or both. Considering a library is a set of modules, a package bundles a set of modules with related metadata (such as name, version and dependencies) which can be distributed as a standalone unit.

Modularization was represented in ROOT Planning Meeting 2017 as a set of separate deliverables:
1. Essential for opening ROOT to collaborators willing to contribute with a large chunk of functionality 
2. Develop a model for building, distributing and deploying ROOT modules that can extend an existing installation of ROOT on demand
3. Evolve ROOT into `BOOT` (à la R) 
4. Package the current ROOT TFile plugins as ROOT modules 
5. Cope with package/module dependencies by interfacing ROOT with popular package managers (e.g. Spack, Conda, …)”

## Problem Statement

Modularization defines a way of grouping of functionality from a software product. It outlines groups in form of modules which identify a particular piece of functionality to solve a set of problems. In general, modularization helps reducing management, coordination and development costs.
It is challenging to define the best set of mechanisms to be able to provide efficient reusability and in the same time modularity of ROOT. Our task is to minimize this cost for ROOT modularization project.
In subsection below “Investigation: ROOT graph of library dependencies” is defined by a big and complex relationship diagram. Better boundaries between layers help it scale as a project, the level of expertise for the contributor needs can be more localized.
By making the boundaries explicit through modules, we can better define and implement a “minimal ROOT,” increasing the chances its functionality can be embedded in other contexts.  This helps ROOT users interact with the wider data science ecosystem.
In the same time, packages and package management provide a mechanism for ROOT users to socialize and and reuse projects built on top of ROOT, it helps to make ROOT more flexible and open for new customers.  This allows ROOT to continue to serve as a community nexus.
In particular, this provides the ROOT team with an improved mechanism to say “no” to new modules within the ROOT source itself as users can simply share their packages.
Having a package format can help define community standards and allows the community to go in a similar direction, focusing what effort we have and then in the same time to reduces the desire / ability for every experiment and analysis group to ‘invent their own’ set of packaging on top.

## Proposed solution

Main idea is to provide modularisation of ROOT and easy separability in multiple discrete and independent modules. Packaging system consisting from ROOT package engine and ROOT package manager, with help to get rid of complex library dependencies that are introduced in current product status.

## Idea


Minimal requirement for ROOT package manager is  to be able to define the dependency and the version, in manifest of package. It could be done either for ROOT packages or for external dependencies (compression algorithms packages for example). This should be a basic schema for distribution packages that are stored in  https://github.com/ and defined as dependencies.
Maximum requirements for package manager could be a next set of features (the list is inspired by functionality of Swift Package Manager, which is supported by a large Swift community):
1. Automated Testing
2. Cross-Platform Packages
3. Support for Other Build Systems (brew and etc.)
4. Support for Version Control Systems
5. Library-Based Architecture
6. Standardized Licensing (CMS related problem)
7. A Package Index
8. Importing Dependencies by Source URL
9. Module Interdependency Determination
10. Dependency Resolution
11. Resource Management

## Source compatibility

## Alternatives considered
One of possible proposals was to introduce a "TInstaller" which could manage access to dependencies, subpackages and be able to install missing parts. Alternative approach was to provide a “homebrew” like tool that will be able to provide cross platform installation of modules (libraries), with dependencies based on recipes. The main idea is to try to have a minimal installation of ROOT (libCore, libCling, libRio..) and running session from prompt, while calling a missing in installation ROOT object or function, will be proposed possibility to install missing component and from the current session (without changing it) to use it. “Homebrew” packaging system by itself is Git-based system with allow user or developer easily upload, synchronize, and extend existing modules - recipes. Homebrew is supported on Mac OS X, has a fork for Linux - LinuxBrew, and there are some efforts in porting it to Windows (need to be tested). Homebrew by itself already has a recipe of ROOT version 5.34.36 and 6.08.10, so first step could be to update existing recipe for ROOT6 and other tags. The second alternative is to develop "ROOTBrew" system.


## Packaging
One of the possible examples of ROOT package is abstract “Math” package, that could consist of multiple math related modules depending on package vendor.
To have a successful socialization of ROOT project via modularization, we need to agree on a format of ROOT package and define set of available modules and its packages for existing monolithic ROOT framework.

### Example of package
Main problem is be defined is what will be a format of ROOT package and how to resolve it.
The most convenient format is a zip file  with manifest that could define a way how the package should be plugged into build system.
Zip file format is also friendly format for Github distribution of packages which making it more friendly for external developers and contributors.

```
dependencies: 
      [Package.Dependency] = [.package(url: "https://github.com/cms-externals/zlib.git", .exact("1.2.11"))]

package:
    name: "ZIP",
    products: [
        .library(name: "ZIP", targets: ["ZIP"])
    ],
    dependencies: dependencies,
    targets: [
        .target(name: "ZIP"),
        .testTarget(name: "ZIP", dependencies: ["ZIP"])
    ]
```
To add ZIP package as a dependency Core,  it should be added  to the Dependencies of Core Manifest file and refer to that dependency in Target.

### Manifest file
Package is defined by set of fields defined in “manifest” file.
Manifest file is particularly defined the format of file that defined the invoice of future properties of package. Manifest need to be well structured to allow tools to automatically modify the manifest. 
Contents of a package:
1. Manifest: set of metadata describing the package.
  1. Name of package
  2. Version
  3. Versioned dependencies of package
  4. Possible source URL
2. Data:
  1. Set of entities which are grouped in a module
3. Tests
4. Docs

## Versioning


## Dependencies
There is no single centralized index of packages. A package's metadata and dependencies are specified in manifest file, which is stored along with the code in its repository. 

### "Dependency Hell"
When a project's packages have requirements that conflict with one another, it creates a situation defined as "dependency hell".  Dependency hell is a common problem found in software that are built using an add-on software package or that rely on one for complete functionality. Dependency hell can take many forms and occur for many reasons, such as the need to instal add-on software libraries, the need for long chains of installations, problems with a conflicting program, the creation of circular dependencies and more.

## Resolution

## Detailed design and Aspects of the Design

### A Build System for ROOT Packages

### Convention-Based Configuration

### Manifest file Format

### Explicit Dependency Declaration

### Packages and Modules

### System Library Access with Module Maps

### Semantic Versioning

### Build Environment Isolation

### Source-Based Distribution

### Package Decentralization


## Features

### Automated Testing

### Support for Other Build Systems

### Support for Other Version Control Systems

### Library-Based Architecture

### Standardized Licensing

### A Package Index

### Dynamic Libraries

### Enforced Semantic Versioning

#### Importing Dependencies by Source URL

### Module Interdependency Determination

### Dependency Resolution

### Resource Management






