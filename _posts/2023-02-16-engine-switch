---
layout: post
title:  "Goodbye, MonoGame..."
summary: "At least for a while."
author: Lajbert
date: '2023-02-16 10:59:23 +0530'
category: update
#category: ['jekyll', 'guides', 'sample_category']
#thumbnail: /assets/img/posts/platformer2.gif
keywords: c#, csharp, monogame, 2d, game, development, video game
permalink: /blog/mac_os_issues/
usemathjax: true
---

If you've been following my work, you know that I had a passion project called Monolith Engine, a free, an open source, 2D engine which I wanted to use my release my games to mobile and console platforms. 
While initially, MonoGame framework seemed like the perfect solution to build my game engine on, after more than one year spent developing my engine, I had to realize that there are too many caveats that I could not have forseen. After long nights on thinking and careful consideration, I've come to the painful conclusion that I have retire Monoloth Engine, at least until MonoGame gets into a state where most of the following risks and issues I consider unacceptable are addressed.
The blog posts and source code will stay available in their usual places, but they are marked as 'archive' and will not receive further updates. Also, further on, unless MG issues are addressed on the long run, there will not be any new posts about MonoGame framework or Monolith Engine, but about the engine I'm using.  
The reasons why I switched from MonoGame and MonolithEngine:
* In theory, it's cross-platform, but I see forum posts of people desperately trying to reach the developers to gain access to the console specific SDKs, without luck, even though they are licenced for the target platform. I've also seen developer whose publisher deal got jeopardized by not being able to get it touch with MG team about the SDK. The fact that such a small team maintains the framework and has excluse control over the SDk imposes a risk that I don't want to take (what happens if the abandon the project, or something happens to them?)
* At the time of writing this post, I'm still unable to make MonoGame 3.8.1 work on my Mac, and I'm not alone with that.
* I have constant issues on Mac with .Net and other dependencies. Since MonoGame is not self contained, it takes a lot of struggle to everything working on a Mac, and the MG updates tend to break everything that was working so far.
* Apple will eventually decommission OpenGL. Nobody knows when, but I really don't want to end up in a situation when I'm about to finish my game, just to realize I won't be able to release it iPhone because it uses OpenGL. MonoGame currently does not have a clear vision of how to address this.

Considering the above, I've decided to look around the landscape again, experiment a with other engines and frameworks, and decided to use Godot engine for now. Using an engine is definitely not as fun as writing your own, but Godot does not feel bloated, is being actively developed, and Godot 4 will support Vulkan, which can allow games to be released of iPhone using MoltenVK even after Metal enforcemnet took place.
Let's see how my first game goes with Godot, and then decide whether to stick to it or look for another option.

Happy coding!