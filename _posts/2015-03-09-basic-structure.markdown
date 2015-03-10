---
layout: post
title:  "Basic Structure"
date:   2015-03-09 13:57:14
categories: integration
summary: Getting started. Embedding Haskell in Visual C++ as a Plugin for Unreal Engine 4.
---

Filthy filthy filthy, thats how this is going to work at first. The goal here is to get Unreal Haskell to trigger computation in Haskell from Blueprints, with glue coming from the plugin.

So there is a separation. The Haskell being called from Blueprints must NOT be a part of Unreal Haskell, but rather a sample consumption of the plugin. Ensure the consuming code is as removed as possible from the gluing code has to be there from the very beginning.

## Just Haskell and Visual C++

Before getting to deep, I thought it was worth it to do a trail run and get Haskell to embed in Visual C++. You can see the results of that exercise here:

[github/fresheyeball/haskell-visual-cpp-minimal](https://github.com/Fresheyeball/haskell-visual-cpp-minimal)

Because `Main` will be in C++, the Spine-less Tag-less Graph Machine (STG) must started and stopped inside C++. To do this there are a few challenges to overcome:

- GHC comes bundled with GCC, so compiled Haskell binaries cannot be seamlessly included by Visual C++
- Haskell's FFI is oriented around C not C++ and the specific Start and Stop functions can't be included in C++
- Documentation is sparse

### Luckily resolving these is not overly complicated or challenging.

The GCC vs Visual C++ compiler gap can be resolved by packaging GHC/GCC binaries inside a `.DLL` with a `.Lib` and including the .Lib as an "Additional Dependency" in Visual Studio.

Getting the Start and Stop functions for STG is resolved by producing a simple C file that can be compiled together with the Haskell Binaries in producing the DLL. You can find that file [here](https://github.com/Fresheyeball/haskell-visual-cpp-minimal/blob/master/CPPHaskellSimple/HsStartEnd.c). Various versions of this file exist around the interwebs, I've no idea who to credit with its creation.

```C
#include <Rts.h>

void HsStart()
{
   int argc = 1;
   char* argv[] = {"ghcDll", NULL}; // argv must end with NULL

   // Initialize Haskell runtime
   char** args = argv;
   hs_init(&argc, &args);
}

void HsEnd()
{
   hs_exit();
}
```

Its now possible to call Haskell from Visual C++, starting and stopping STG with an `extern`.

```C++
extern "C" {
  void HsStart();
  void HsEnd();
}
```

Now all that is needed is to include the Header file outputted by GHC and call the functions outputted from Haskell `ccall`.

```haskell
foreign export ccall <function name> :: <function type>
```

## Plugin Structure

The basic structure of an Unreal Plugin is well templated [here](https://wiki.unrealengine.com/Plugin,_Functional_Code_Template_For_You) by the immortal and mighty [Rama](http://evernewjoy.net/).

```haskell
Project Root
│
├─ Binaries
├─ Build
├─ Config
├─ Content
├─ Intermediate 
├─ Plugins ◄-- Create this folder
│  │
│  └─ MyPlugin ◄-- Then just drop in your plugin
│     │
│     ├─ Binaries
│     ├─ Resources
│     ├─ Source
│     │  │
│     │  ├─ Private 
│     │  ├─ Public
│     │  └─ MyPlugin.Build.cs ◄-- This runs before C++ is compiled
│     │  
│     └─ MyPlugin.uplugin ◄-- Basic plugin config
│
├─ Saved 
├─ Source
│
├─ MyProject.sdf
├─ MyProject.sln
└─ MyProject.uproject
```

And it is a folder as in a directory as in it is NOT a Visual Studio filter (a weird thingy that looks like a folder).

## Unreal Build Tool

So Unreal Engine uses a build system called the Unreal Build Tool (UBT) which runs on Mono in C#. It seems to handle linking and some code generation, orchestrating the build process until the Unreal Editor launches and takes over from there. I'm sure it does much more than that, but for wiring in Haskell thats all that should be necessary. 

UBT files are in C# and named in the convention `*.Build.cs`. To get Haskell compilation to work with UBT it just needs to spawn GHC compilation and DLL packaging from inside of the plugin's `Build.cs`. UBT will also grab anything that looks even remotely related to C in the `Source` folder, so packaged DLLs, Headers and such will need to go there, but in the case of Unreal Haskell, not the Source code. 

This is because the plugin source will need to contain files like `HsStartEnd.c` which are intended for GHC compilation not UBT slurpage. As well the binaries we hand UBT should contain logic that ultimately comes from the consumer, not the plugin itself.

## So far..

```haskell
Project Root
│
├─ Binaries
├─ Build
├─ Config
├─ Content
├─ Intermediate 
├─ Haskell ◄-- Plugin consumer code goes here
├─ Plugins
│  │
│  └─ UnrealHaskell
│     │
│     ├─ Binaries
│     ├─ Resources
│     ├─ PackageGHC ◄-- Stuff in here will grab the consumer code and build
│     ├─ Source
│     │  │
│     │  ├─ Private 
│     │  ├─ Public ◄-- Resulting binaries C++ and Headers show up here
│     │  └─ UnrealHaskell.Build.cs ◄-- Starts the Compilation
│     │  
│     └─ UnrealHaskell.uplugin
│
├─ Saved 
├─ Source
│
├─ MyProject.sdf
├─ MyProject.sln
└─ MyProject.uproject
```

Once this is proven out round trip, its time to do some code generation and have glue written at build time.