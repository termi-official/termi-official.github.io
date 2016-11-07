---
layout: post
title:  "Lets introduce the resource management system!"
date:   2016-02-20 01:00:00 +0100
category: game-engine-technologies
section: Game Engine Technologies
---


The resource manager is a crucial part for the rise or fall of game engines. It determines how
flexible, pluggable and integrative the engine is. More important, is it a central point in
performance.

So lets talk about what the resource manager's job is. And equally important: lets talk about what
it is not!


What it has to do....
- Packaging resources into groups
- Manage ownership and dependencies
- Determine when to load, unload and stream a resource
- Determine memory bounds


And what not!
- How to load resources
- Where to load (GFX mem, main mem, ... )
This is part of the specific subsystem.
