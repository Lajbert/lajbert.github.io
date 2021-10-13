---
layout: post
title:  "Project setup for cross-platform game development in MonoGame"
summary: "It makes life so much easier..."
author: Lajbert
date: '2021-10-08 18:14:23 +0530'
category: update
#category: ['jekyll', 'guides', 'sample_category']
#thumbnail: /assets/img/posts/platformer2.gif
keywords: c#, csharp, monogame, 2d, game, development, video game
permalink: /blog/cross_platform_project/
usemathjax: true
---

<a href="https://lajbert.github.io/blog/game_announcement/">As I promised earlier</a>, we are going to take a look at how to develop a game in MonoGame targeting multiple platforms at the same time, using Visual Studio 2019. It may seems like a nuisance first, but once you’ve experienced the pain of deploying a game to mobile that you have originally developed to PC (or some other platform), you’ll see why it’s so great to be able to run it on both platforms and catch the performance and other issues early on, rather than seeing all the problems at once, not even knowing where to start fixing/changing the code.

Pre-requisite for this tutorial: have MonoGame and Visual Studio 2019 installed according <a href="https://docs.monogame.net/articles/getting_started/0_getting_started.html">MonoGame’s Getting Started documentation</a>. And, of course, a bit of MonoGame knowledge is also helpful. If you lack the basics, go through <a href="http://rbwhitaker.wikidot.com/start">RB Whitaker’s very good tutorials</a>, and you’ll be up to speed in no time.

This is approach is also very useful even if you are only targeting mobile, because running and testing your game on your computer with mouse+keyboard is much faster than always deploying the whole package to your phone, especially when you make frequent, small code changes or finetuning some features that requires lots of consecutive deployments. So let’s get to it!

There is no superior, streamlined way to develop a game, so we are going to take a look at my approach, which means the following:

1. I have a <a href="https://github.com/Lajbert/MonolithEngine">video game engine</a> that I’m using to make my game(s), this a shared code in a separate project
2. I have a game project, that uses the aforementioned engine code from project 1. and contains the game logic itself, a “platformless” shared project. This code can’t be ran in itself, but it can be included in the target platforms’ solutions.
3. I have 2 separate projects targeting mobile and PC, both includes the code from project 2., but they each run the game on the respective platform.