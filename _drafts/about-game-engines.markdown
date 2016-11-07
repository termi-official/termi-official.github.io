---
layout: post
title:  "Oh, another blog about game engines?"
date:   2016-02-10 01:00:58 +0100
category: game-engine-technologies
section: Game Engine Technologies
---

For sure another one! But this blog will be different. I will focus on infrastructure, architecture
and theory. Especially on how to collaborate instead of going into isolation.


# Small overview of some game engines to analyze

If you want to look for some specific patterns, architecture or implementation (techniques), just
check it out on the engine of your choice in the language of your choice. Many engines are
developed in C or C++ since these kind of 'lower level' languages gives the developers more
control over the code. Depending on what kind of game or engine you want to design JavaScript,
Haxe or C# or whatever language you want may be sufficient.

- [Unreal Engine 4](https://www.unrealengine.com/what-is-unreal-engine-4)
- [Urto3D](https://urho3d.github.io/)
- [ID Tech 4](https://github.com/id-Software/DOOM-3)
- [NeoAxis](https://github.com/NeoAxis/SDK)
- [Godot](https://github.com/godotengine/godot)
- [Atomic](https://github.com/AtomicGameEngine/AtomicGameEngine)
- [Banshee](https://github.com/BearishSun/BansheeEngine)
- [Spring](https://github.com/spring/spring)
- [Panda 3D](https://github.com/panda3d/panda3d)

For a complete overview on game engines look at [devmaster](http://devmaster.net/devdb/engines) and
on [github](https://github.com/showcases/game-engines).


# Overview of some components to analyze
No chance to get them all, so I will only list the ones I have experience with.

- [BulletPhysics][bullet-home]
- [Havoc][havoc-home]
- [GameWorks][gameworks-home]
- [GpuOpen][gpuopen-home]
- [eastl][eastl-git]
- [LUA][lua-home]
- [OGRE 3D][ogre-home]
- [bgfx][bgfx-git]



# Related work
I also attach a small list with pages which discuss or explain different game engine related
topics.

- [gamedev.net][gamedev-home]
- [Jorge "BSVino" Rodriguez][game-math-channel]
- [Randy Gaul][randy-gaul-home]
- [CppCon](https://www.youtube.com/user/CppCon)
- [Writing a Game Engine from Scratch][game-engine-from-scratch-gamasutra]
- [FutureProofGames][future-proof-games]
- [GDC State](http://www.gdconf.com/conference/programming.html)
- [Molecular Matters][molecular-matters-home]
- [Game Programming Patterns](http://gameprogrammingpatterns.com/)


# What is wrong with current engines?

The main problem is: even the open source engines work against each other! Everyone who ever had
developed middleware for games knows how hard it is and how much time it takes to write a plugin
for every bigger game engine you want to support. Or how hard it is as a engine developer to
get some middleware into the engine. There is simply no standardization or consensus.
An example of working standardization on plugins/middleware is common in the audio industry. There
is a thing called [VST][vst-spec] (Virtual Studio Technology), which specifies the communication
between the Digital Audio Workstation (or any audio software system type of your coice) and the
middleware. It simplifies the software creation process so much and this finally comes down as a
benefit for the user! So lets take this approach as a role model. Additionally we measure the
protocol's overhead for game engines to approximate how big the performance hit will be.




# Methodology

In these articles I going to get the theory first. To prove them I will get some benchmarks
running. To design them I will use different stochastic methods. Additionally I going to analyze
existing (open source) games extremely detailed. The reason I stick to open source is simply
because everyone can access the code, so the results stay verifiable and reproducible. Or in
other words - I try to be as scientific as possible. The measurements itself will take time and
memory complexity in account, as well as cache coherence and architectural overhead.


<!--
- game development patterns
- Profile existing games -> Benchmarking framework
- STL + https://github.com/foonathan/memory vs EASTL
-->

# Requirements Analysis

So we start with some basic questions. What are the major systems in current game engines?
A good overview is shown in [game engine architecture][game-engine-book] by Jason
Gregory.
I will answer the question with an analysis of what game engines are needed for. The naive answer
is: To create games. A more technical answer: An engine is a set of tools and interconnected
frameworks to support people creating games. So lets take a look on basic architectures of how
things are designed within existing engines.

Lets start with game objects - there are 2 major approaches out to get the game objects together.
The first one is to keep them classical object oriented and simply specialize the classes when
a new kind of object is needed. [Ray Wenderlich][wenderlich-ecs] has shown why this approach is
terrible. An alternative is to go data oriented and design game objects as sets of components.
Since game objects use a wide horizontal inheritance and no deep vertical inheritance like in
classical object oriented cases, this approach is beneficial. The according design pattern is
called 'Entity-Component-System'. Another huge benefit is a better cache utilization.

<!-- TODO: image with horizontal vs vertical inheritance -->

It is possible to describe mostly any video game as a [series of states][game-state-proof]. For the
rest of the articles I will introduce the definition of a game state as exactly the current
active state within the global state machine. The next big part is the so called "game loop", which
is a set of transformations to update the current game state. This loop can be either serial,
parallel or asynchronous. Modern computer systems offer multi-core architectures, so a serial is
not interesting at all. Lets focus on the sync vs async loops. Parallel loops are utilized via so
called job systems. These systems provide a directed - acyclic graph of interdependent tasks to
process, whose result is the newest game state. On the other hand there are the async loops which
update shared states interdependent from concurrent systems.

>TODO: Image sync vs async  
>TODO: Link to an article

Now that the basic things about how to run the engines are resolved, we can focus on how to manage
components. This automatic managing is called resource system and determines how flexible an engine
is going to be. It also is a key part for better performance. The significance comes from its
nature, since it loads and organizes the big chunks of data on disk and in memory. Redundant data
in memory can result in critical performance loss. A bad interface design can make the whole
process terrible slow, from the runtime as from the production perspective.

>TODO: Link to an article

Well, this already is the overview about the most important pieces. Two additional systems are the
game editor and the toolchain. I am going to handle these systems in separated articles since they
are related to game engines, but more the kind of systems on top like an IDE for programming
languages.

For completeness I entitle the remaining minor pieces. These are some kind of operating system
abstraction, if you want to develop cross platform. Then there is a virtual file system to load
and store data on storages. Also one should not forget the math library. If the language of choice
does not provide concurrency by default a threading library is necessary. Additionally it is
always helpful to get a logging or even journaling library to track down errors and problems
faster. Finally there should be some unified memory management or at least coarse memory tracking
for debugging and optimization purposes.

Engine (sub)systems are growing in complexity, so they should be separated. The community already
started with the physic engines as separate parts. We should start to collaborate instead
of going into isolation, everyone with his own engine completly from scratch! I think the scene
suffers from some kind of [Not-Invented-Here-Syndrome][nih-syndrome], since I really do not see any
components getting reused. In common many people say, existing frameworks do not offer enough
performance, so I will going to profile everything. Some other people argue, that existing
frameworks do not offer the functionality they need, while their implementations bloat up classes
instead of separate them properly by functionality ([Single responsibility principle][single-responsibility-principle]). There is absolutely
[no good reason for god objects][god-object].

This was already the overview on what game engines do. There are many many parts I had not
mentioned in this article, for example graphics, physics and audio. These are implicit parts of
game engines and should ideally be modular, swappable systems. Some games need only 2D graphics
and 2D physics with high performance networking while other games need 3D graphics and 3D physics
with high performance networking - so we end up with permutations of systems. Another implicit part
of game engines is the game editor, which I will analyze separately since it is not explicitly
needed and should also be swappable, too. Ideally the communication is managed via network
connection and (another) specified protocol. This separation arises on the one hand
from the fact, that the editor and the target machine are in some cases 2 different devices, while
on the other hand the code base gets more manageable.



[eastl-git]: https://github.com/electronicarts/EASTL
[game-engine-book]: http://gameenginebook.com
[nih-syndrome]: https://en.wikipedia.org/wiki/Not_invented_here
[bgfx-git]: https://github.com/bkaradzic/bgfx
[dev-ops-in-game-dev]: http://futureproofgames.com/blog/2015/11/16/devops-game-dev-beginning/#.Vr53HkJuM6s
[molecular-matters-home]: http://blog.molecular-matters.com/

[gamedev-home]: http://www.gamedev.net/page/resources/_/technical/
[game-math-channel]: https://www.youtube.com/user/BSVino/playlists
[randy-gaul-home]: http://www.randygaul.net
[game-engine-from-scratch-gamasutra]: http://gamasutra.com/blogs/MichaelKissner/20151027/257369/Writing_a_Game_Engine_from_Scratch__Part_1_Messaging.php
[future-proof-games]: http://futureproofgames.com/

[ogre-home]: http://www.ogre3d.org
[gpuopen-home]: http://gpuopen.com
[gameworks-home]: https://developer.nvidia.com/gameworks
[havoc-home]: http://www.havok.com/
[lua-home]: http://www.lua.org/
[bullet-home]: http://bulletphysics.org/

[vst-spec]: http://www.steinberg.net/en/company/technologies/vst3.html

[wenderlich-ecs]: http://www.raywenderlich.com/24878/introduction-to-component-based-architecture-in-games
[single-responsibility-principle]:http://www.oodesign.com/single-responsibility-principle.html
[god-object]:http://blog.decayingcode.com/post/anti-pattern-god-object
[game-state-proof]: http://www.gamedev.net/page/resources/_/technical/game-programming/brain-dead-simple-game-states-r4262
[online-game-engines-article]: http://www.gamedev.net/page/resources/_/technical/apis-and-tools/online-video-game-engines-discover-why-you-should-use-them-r4295
