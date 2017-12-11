# rootcling refactoring

* Proposal: [RE-0003](0003-root-migrate.md)
* Authors: [Raphael Isemann](https://github.com/Teemperor)
* Review Manager: TBD
* Status: **Awaiting review**

## Introduction

rootcling is a tool of ROOT that generates files that are needed for ROOT's runtime, such as dictionaries or `rootmap` files. This proposal discusses how rootcling could be modernized to make it easier to adapt to new requirements and to maintain.

root-evolution thread: [Discussion thread topic for that proposal](https://root-forum.cern.ch/c/root-evolution)

## Motivation

rootcling has a central role in the ROOT ecosystem as it generates a wide variety of essential files for ROOT's runtime environment. It is also a part of ROOT that needs to evolve with ROOT itself, as many major changes to ROOT's behavior will also require changes in rootcling. For example with the introduction to C++ modules, a significant amount of time was spent on allowing rootcling to also produce the required C++ module files.

This central role should make rootcling a prime target for a part of ROOT that has good code quality, testing and well-crafted requirements. However, in reality rootcling isn't as well-tested and well-maintained as it should be.

rootcling has no dedicated maintainer that has an overview over the whole functionality, mostly because the requirements of rootcling encapsulated a wide variety of tasks that are necessary to generate all needed output files. This makes it hard for a single person to understand the whole architecture of rootcling and to give reliable code review.

rootcling also doesn't have the best code quality. It has undocumented functions that take dozens of undocumented arguments which are then processed in the function body spanning thousand of lines. Some classes like RScanner violate const-correctness, some functions just do mostly the reverse actions that their calling functions do and there are many undocumented magic numbers manipulating pointers to C-style arrays.

#### The C++ modules problem

These shortcomings became very apparent after the introduction of C++ modules to ROOT: Suddenly rootcling, which now also had to generate a C++ module alongside the other files, needed to respect dependencies between headers.

This meant that rootcling invocations couldn't be run in serial anymore and not running rootcling in the right order would produce errors. Also the interpreter runtime helpers (like `input_line` source locations) that already exist in the PCH were now duplicated in all produced C++ modules. Furthermore, implementing the C++ modules generation from inside rootcling (which also means from inside the cling interpreter), requires us to duplicate a lot of work that clang could do for us.

The obvious solution to this is not implementing the C++ module generation inside rootcling. However, due to all the shortcomings of rootcling's design, it's not possible to simply extract the C++ module logic into its own independent executable and leaving rootcling unaffected. A C++ module created in any other way than reusing the rootcling infrastructure will cause unexplainable and hard to debug test failures in ROOT, as it could miss some part that rootcling somehow added to a normal C++ module.

To summarize: The current situation of rootcling is not good for the development of ROOT. Any future features to rootcling require enormous amount of work to correctly implement them.

## Current architecture

### Bootstrapping

### Input files

1. ##### Header files

   rootcling takes several headers that it will parse. These headers are supposed to contain all the declarations that we need for IO and autoloading.

2. ##### LinkDef

   rootcling also takes a LinkDef.h, which defines what goes into a dictionary. This is documented [here](https://root.cern.ch/selecting-dictionary-entries-linkdefh).

### Output files

rootcling has several output files serving different purposes:

#### 1. rootmap files

The `.rootmap` files contain the class names and headers that are connected with a certain shared library of a dictionary. When those headers or class names are used in ROOT, we automatically load the shared library.

Example rootmap file:

```C++
{ decls }
namespace std {  }
namespace __gnu_cxx {  }
namespace __gnu_cxx { template <typename _Iterator, typename _Container> class __normal_iterator; }
class TString;
namespace std { inline namespace __cxx11 {  } }
template <typename AParamType> class TParameter;
namespace ROOT {  }
namespace ROOT { namespace Detail {  } }

[ libCore.so ]
# List of selected classes
class ColorStruct_t
class CpuInfo_t
class Event_t
class FileStat_t
class GCValues_t
class MemInfo_t
class PictureAttributes_t
//[stripped file for demonstration purposes]
```

#### 2. ROOT PCM

The  `_rdict.pcm` files contain memory layout information of classes (deduced from the specific class declaration and its members) so that this information is accessible to ROOT without the need of loading the specific class into the interpreter. They are actually ROOT files, not clang's **p**re**c**ompiled **m**odules, but they reuse the same file extension with an additional `_rdict` before the extension.

As they are only an optimization to prevent expensive header parsing just to get the memory layout of a class, they could be replaced by the C++ module if the loading from the C++ module and regenerating the IO information from there is not much slower than loading from the ROOT PCM.

Note that many ROOT PCMs for ROOT modules are actually empty and their information is actually stored inside the ROOT PCH. You can check this by seeing who is passing `-writeEmptyRootPCM` to rootcling when building ROOT.

#### 3. C++ modules PCM

The C++ modules containing all the headers of the module precompiled as a clang PCM (**p**re**c**ompiled **m**odule). 

#### 4. `G__source.cxx`

Contains code that is compiled into the shared library of the ROOT module. This shared library is loaded when ROOT find a class or a header that is mapped via a `.rootmap` file to it. It contains special constructors for the selected classes (see LinkDef.h), implemented to the functions declared by ClassDef() and a dictionary initialization routine. This routine will be executed when loading and informs ROOT about the include paths and the list of forward declarations that map decls to specific headers.

#### 5. The ROOT PCH

The ROOT PCH is a clang **p**re**c**ompiled **h**eader that contains all the precompiled headers of certain core ROOT modules. The PCH is technically very similar to a C++ module PCM and is just a file from which ROOT can load on demand certain parts of the code that it needs (like class or function definitions).

## Possible improvements

### Splitting up rootcling by functional requirements

One way to make rootcling easier to understand is splitting up the responsibilities for generating each file into it's own part of the code. The parts can still be linked together to form the original monolithic rootcling invocation, but in theory they should be as independent as possible from each other. 

#### `root_cxxmodule_gen`  generates a C++ module

This part is supposed to generate the clang C++ module file. It parses all the header that it is given and writes the C++ module to disk.

As ROOT needs during runtime a modulemap file, this part will also generate a modulemap from the given header in case the user hasn't specified an existing modulemap. If the user however has specified an existing modulemap, `root_cxxmodule_gen` will verify that the headers in the modulemap are identical to the headers passed in via the command line arguments. When using CMake, the `ROOT_GENERATE_DICTIONARY` function will take care of generating the modulemap and passing the right headers to the executable.

It is the only part of rootcling that needs to respect the inter-header dependencies of modules. For example, if a header of Module A is referencing a header of Module B, we first need to call `root_cxxmodule_gen` on B and then on A.

#### `root_rootmap_gen`  generates a rootmap file

#### `root_rootpcm_gen`  generates a ROOT PCM

#### `root_io_gen`  generates a G__source.cxx

#### `root_pch_gen`  generates the ROOT PCH

### Code quality


