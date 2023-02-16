---
layout: post
title:  "[Archive] MonoGame issues on MacOSX"
summary: "Just a few things to overcome..."
author: Lajbert
date: '2022-01-18 10:59:23 +0530'
category: archive
#category: ['jekyll', 'guides', 'sample_category']
#thumbnail: /assets/img/posts/platformer2.gif
keywords: c#, csharp, monogame, 2d, game, development, video game
permalink: /blog/mac_os_issues/
usemathjax: true
---

MonoGame supports many platforms, including MacOSX, but there are some traps you can walk into when developing on Apple's platform.

* If you get strange exceptions during compiling, make sure you to remove all shader files from your contents (both the files and the references in .mgcb file itself). If build works without the shaders, it means your project works on Mac, you just have to install Wine to be able to compile the shaders during normal the compilation process.
* MGCB Editor from the repository does not work on Mac. Extract and run <a href="https://github.com/MonoGame/MonoGame/files/7530284/MGCB.Editor.zip">this version</a> of the editor, it should work. However, you can only open, edit and save content files with this, building probably won't work.
* To build contents on Mac, you have to use the command line tools the way it's written on <a href="https://docs.monogame.net/articles/tools/mgcb.html">MonoGame's website</a>.
    1. run 'dotnet tool install -g dotnet-mgcb'
    2. once it's ready, cd into your contents folder, and you can use the command '~/.dotnet/tools/mgcb /rebuild Content.mgcb' (or however your .mgcb file is called) to build your contents for the project
    3. Once the contents are built, you can compile and run your project without any issues

If you are developing on a Mac, chances are you want to run your game on an iPhone. If you experience UI freezes on the phone, look at <a href="https://lajbert.github.io/blog/get_data_ios/">this article</a> for a possible solution.

Happ coding!