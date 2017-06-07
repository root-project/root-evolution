# Feature name

* Proposal: [RE-NNNN](NNNN-filename.md)
* Authors: [Author 1](https://github.com/phsft-bot), [Author 2](https://github.com/phsft-bot)
* Review Manager: TBD
* Status: **Awaiting review**

*During the review process, add the following fields as needed:*

* Decision Notes: [Rationale](https://root-forum.cern.ch/c/root-evolution), [Additional Commentary](https://root-forum.cern.ch/c/root-evolution)
* Bugs: [ROOT-NNNN](https://sft.its.cern.ch/jira/projects/ROOT/issues/ROOT-NNNN), [ROOT-MMMM](https://sft.its.cern.ch/jira/projects/ROOT/issues/ROOT-MMMM)
* Previous Revision: [1](https://github.com/root-project/root-evolution/blob/...commit-ID.../proposals/NNNN-filename.md)
* Previous Proposal: [RE-XXXX](XXXX-filename.md)

## Introduction

A short description of what the feature is. Try to keep it to a single-paragraph "elevator pitch" so the reader understands what problem this proposal is addressing.

root-evolution thread: [Discussion thread topic for that proposal](https://root-forum.cern.ch/c/root-evolution)

## Motivation

Describe the problems that this proposal seeks to address. If the problem is that some common pattern is currently hard to express, show how one can currently get a similar effect and describe its drawbacks. If it's completely new functionality that cannot be emulated, motivate why this new functionality would help ROOT developers create better ROOT code.

## Proposed solution

Describe your solution to the problem. Provide examples and describe how they work. Show how your solution is better than current workarounds: is it cleaner, safer, or more efficient?

## Detailed design

Describe the design of the solution in detail. If it involves new class in the framework, show the additions and changes to the ROOT's API. If it's a new API, show the full API and its documentation comments detailing what it does. The detail in this section should be sufficient for someone who is *not* one of the authors to be able to reasonably implement the feature.

## Source compatibility

ROOT's source compatibility requirements for ROOT 6.12 are very strict: we should only break source compatibility if the ROOT constructs were actively harmful in some way, the volume of affected ROOT code is relatively small, and we can provide source compatibility and migration.

Will existing correct ROOT 6.10 or ROOT 6.12 applications stop compiling due to this change? Will applications still compile but produce different behavior than they used to? If "yes" to either of these, is it possible for the ROOT framework to accept the feature in ROOT 6.10 compatibility mode? Is it possible to automatically migrate from the old syntax to the new syntax? Can ROOT applications be written in a common subset that works both with ROOT 6.10 and ROOT 6.12 to aid in migration?

## Effect on ABI stability

Does the proposal change the ABI of existing language features? The ABI comprises all aspects of the code generation model and interaction with the ROOT runtime, including such things as calling conventions, the layout of data types, and the behavior of dynamic features in the ROOT (reflection, interpreter, etc.). Purely syntactic changes rarely change existing ABI. Additive features may extend the ABI but, unless they extend some fundamental runtime behavior (such as the aforementioned dynamic features), they won't change the existing ABI.

## Effect on API resilience

API resilience describes the changes one can make to a public API without breaking its ABI. Does this proposal introduce features that would become part of a public API? If so, what kinds of changes can be made without breaking ABI? Can this feature be added/removed without breaking ABI? For more information about the resilience model, see the
[TODO](...).

## Alternatives considered

Describe alternative approaches to addressing the same problem, and why you chose this approach instead.

