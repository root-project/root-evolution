# root-migrate

* Proposal: [RE-0002](0002-root-migrate.md)
* Authors: [Raphael Isemann](https://github.com/Teemperor)
* Review Manager: TBD
* Status: **Awaiting review**

## Introduction

root-migrate is an tool for automatically migrating C++ user code that is based on one version of the ROOT API to a new version, which should make the evolution of ROOT easier for both developers and users.

root-evolution thread: [Discussion thread topic for that proposal](https://root-forum.cern.ch/c/root-evolution)

## Motivation

ROOT’s strive for stable interfaces (API) is partly because millions of lines of source code are depending on them. However, those interfaces still need to evolve towards better code quality and new features. The current conservative development approach is necessary because large projects can’t easily migrate from one API to another.

An automatic process for upgrading code that doesn't strain the human resources of the experiments would diminish the negative effects of changing the ROOT API between versions and give ROOT developers more flexibility.

## Proposed solution

root-migrate is a tool for automatically updating C++ user code. It takes user code, finds any usages of old ROOT API's and then automatically replaces any found code pieces with the correct code for using a newer ROOT API.

A prototype implementation is available [here](https://github.com/Teemperor/root-migrate).

## Detailed design

root-migrate works in four phases which are executed in this order when the user invokes the tool:

1. Parse the user code into a clang AST.

2. Find old usages of the ROOT API in this AST.

3. Update the old usages that were found.

4. Write back to the code to the file system.

root-migrate uses the clang infrastructure to work with C++ code. The reason for this are:

* ROOT is already using clang internally for it's interpreter and so it would be possible to integrate it into the interpreter framework.

* similar projects already use clang's infrastructure with great success (e.g. clang-tidy).

#### 1. Parse the user code into a clang AST.

The first step is transforming the user code into a clang AST that is used by the subsequent steps. To produce this AST root-migrate needs same information that clang needs for parsing code (e.g. used C++ standard, macro definitions, include paths and other compiler flags). To provide the arguments to the tool, the user should provide a [compilation database](https://clang.llvm.org/docs/JSONCompilationDatabase.html) to root-migrate. The compilation database format has the advantage that it is:

* already supported by clang tooling out of the box.

* can be generated in all CMake based projects with `-DCMAKE_EXPORT_COMPILE_COMMANDS=On`.

* simple to generate from custom build systems.

It is likely that the build systems of most experiments already contain an option to generate a compilation database, so generating this file shouldn't bring any additional workload to users. Even if an experiment doesn't have such a file, it's for most projects very easy to write the necessary generation code. Generating a compilation database would also allow this experiment to use all other clang tooling on their code base.

As experiment's software contains usually more than one translation unit, root-migrate will parse each translation unit and then perform the following steps on every translation unit.

#### 2. Find old usages of the ROOT API in this AST.

Now the root-migrate has the AST of the user code, it can start looking for old usages of the ROOT API. This happens by using clang's [AST matchers](https://clang.llvm.org/docs/LibASTMatchers.html), which is a language for describing language constructs in C/C++ that follow a certain pattern. An example matcher would look like this:

```lisp
callExpr(callee(functionDecl(hasName("TFile::open")))).bind("call")
```

This matcher would match any AST node that calls a function with the name `"TFile::open"`. Any matched node is then returned to a custom handler which then can perform an action on this node. In our case the handler is root-migrate and the action is replacing the node with the proper call in the newer ROOT version.

For every breaking API change root-migrate needs to be informed by creating such an AST matcher that matches the old usage of the API. The AST matchers are written by the ROOT team and shipped alongside root-migrate, but it should also be possible to just pass your own AST matchers to the tool for custom upgrades.

An AST matcher can be provided as a string to the tool, so changing an AST matcher just means changing a text file and rerunning the tool over the code base.

#### 3. Update the old usages that were found in step 2.

Along with the AST matcher for finding the piece of code, root-migrate also needs the code that it should replace the found piece of code with. This new code is just a string with placeholders for certain parts of the found code by the AST matcher. A pair of an AST matcher and the corresponding replacement text is what we call an **upgrade** from now on.

The following example is the replacement for above's AST matcher. 

```php
$call.release()
```

It will take the found call, and then just append the string `.release()` behind the found code.

#### 4. Write back to the code to the file system (optional).

After the tool has updated the in-memory buffer of the code, it should be written back to disk to make the change persistent. As the new code is no longer able to compile with the current ROOT version, the code should be written back to a temporary file first when multiple translation units are migrated (as another translation unit might read this file later and then is unable to compile the migrated code in there).

### Example

In the following example we use the upgrade from above to change `TFile::open` to return an `std::unique_ptr` instead of `TFile *`. 

Now we take the following code as an example for user code that needs to be upgraded:

```haskell

int main() {
  TFile* F = TFile::open("file.root");
}

```

This code would stop compiling once `open` returns an `std::unique_ptr` . We take our upgrade and run it over the code, which will cause root-migrate to find the `open` call and append a `.release()` behind it to make it return a raw pointer again (and switch back to manual memory management as before).

The file after running root-migrate over it looks like this:

```C++
int main() {
  TFile* F = TFile::open("file.root").release();
}
```

You can find a working example in the `test` directory of the root-migrate prototype [here](https://github.com/Teemperor/root-migrate/tree/2938ac02977c32f2a46cd39b7d52da948ff703b1/tests). Note that in this example we automatically bind the whole match to the `$all` variable, so the `.bind($varName)` at the end is no longer necessary.

### Versioning of requested upgrades

root-migrate requests the user to specify the current ROOT version the code is compiling against and the target version the user wants to update. For example, a user would specify he used version 7.08 before, but now wants to run the code on ROOT version 7.11. This allows the user to migrate from one ROOT version to another without needing the specific root-migrate version.

This also solves the problem of API changes that only differ in semantics . Let's take the following API change:

```C++
// old API:
int func(TObject* toRemove, TObject* toAdd);
// new API:
int func(TObject* toAdd, TObject* toRemove);
// user code:
void doStuff() {
  // is this old or new usage of the API?
  func(new TObject(), new TObject());
}
```

As we can see it's not possible to detect if the given `doStuff` function was targeting the old or the new API of the `func` function. However, if the user specifies that this is written agains the old version of ROOT, we can see that we need to exchange the two function parameters to adapt the code to the new ROOT version.

This also requires that every upgrade in root-migrate needs to be annotated which the specific ROOT version in which the API change occurred.

## Open question

### 1. Code written in other languages

User's have multiple ways to use ROOT beside writing C++ code. One of the official ways to use ROOT
is via Python, which means there is also Python user code that needs to be updated.

Updating this code however is more complicated than C++. There is no reasonable way
to parse Python and reason about it's API calls (especially in an OOP environment). It's an open question how/if users using these technologies can be supported by root-migrate.

## Alternatives considered

* It was originally proposed that root-migrate should work by recognizing certain clang error messages and fixing the source for them. The problem with this approach is that it is unable to migrate changes that don't show up as errors (e.g. reordering function parameters of compatible types) and that it would require to always keep an updated list of clang error messages. Also the fixing process would get indefinitely more complicated when trying to work on an already broken clang AST.

