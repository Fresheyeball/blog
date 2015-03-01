---
layout: post
title:  "Unreal Haskell Motivation"
date:   2015-03-03 13:57:14
categories: integration
summary: Why I'm bothering to hook Unreal's Blueprint's up to Haskell, and what such a structure might look like.
---

So here is the thing. I want to make games. Producing art and code are passions that are deeply personal, and have to be practiced and the zenith of your ability to be satisfying. And as we all should know, flow is extremely important. 

After trying out several platforms, one stood out for the quality of its tooling and resulting games.

## Unreal Engine 4

In practicing UE4 flow was achieved in the level editor, and with simple blueprints; but not business logic. Math in blueprints is insane to read, as are (the equivalent of) switch statements and many other constructs. Blueprints seems to excel at interactive logic chains where the code is describing a response from the player, or performing actions in the world.

The alternative that comes out of the box is C++, which I don't know or have a desire to learn. Flow for me right now is in the purely functional languages and its painful to give that up. C++ also has a pretty high learning curve, so the likely-hood of achieving flow in C++ is more than a little far off.

So there is a hole, a place where business logic needs to live, and where I can have flow. Blueprints stay in the picture because they excel at specific types of logic so what are the properties of Blueprints? 

## Blueprints

They do actually hold some functional characteristics:

- Referential Transparency
- Explicit Purity and Effects
- Literals are Immutable
- State has to be 'unwrapped'

Referential Transparency in particular should make Haskell a good compliment. If there was a way to communicate between Haskell and Blueprints. 

Hopefully that all doesn't sound to crazy.

## Haskell and C++

The Glorious Glasgow Haskell Compiler uses GCC in the compilation process and and comes equipped with a mature Foreign Function Interface to communicate with other binaries. However its still rather painful to use if you intend to call C or C++ from Haskell. Calling Haskell from C++ works well though and can be done more cleanly. 

That, and realistically I wont be able to write complete and well designed bindings to Haskell for the Unreal C++ API. Its too large and undertaking. 

**If you would want to do this, or think there is a way it could be feasible, please contact me.**

So a more feasible application structure would be to have Blueprints own actions on tick and the Engine API calls, while Haskell owns business logic and mathematics. 

So something like this:

```
Blueprints : The level is starting, whats the GameState?
Haskell    : Here you go, a GameState
Blueprints : Hey so X happened, whats the new GameState?
Haskell    : From X? let me think... ok its got to be GameState
Haskell    : Blueprints, respond to this update
Blueprints : Ok, I will effect the world
Blueprints : I am always effecting the world lol
Blueprints : Hey can you do this math? I suck at math...
Haskell    : Sure, here's the answer
Blueprints : You the best calculator ever Haskell
Haskell    : Well you know how to do the real Effects!
```

It should work right? It could even be elegant. 