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

Because `Main` will be in C++, the Spineless Tagless Graph Machine (STG) must started and stopped inside C++. To do this there are a few challenges to overcome:

- GHC comes bundled with GCC, so compiled Haskell binaries cannot be seamlessly included by Visual C++
- Haskell's FFI is oriented around C not C++ and the specific Start and Stop functions can't be included in C++
- Documentation is sparse

## Unreal Build Tool

So Unreal Engine uses a build system called the Unreal Build Tool (UBT) which runs on Mono in C#. It seems to handle linking and some code generation, orchestrating the build process until the Unreal Editor launches and takes over from there. I'm sure it does much more than that, but for wiring in Haskell thats all that should be necessary.

## Plugin Structure

The basic structure of an Unreal Plugin is well templated [here](https://wiki.unrealengine.com/Plugin,_Functional_Code_Template_For_You).