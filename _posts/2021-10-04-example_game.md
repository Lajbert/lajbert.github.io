---
layout: post
title:  "Cross-platform game exampel coming"
summary: "After all, Monolith Engine is cross-platform."
author: Lajbert
date: '2021-06-24 18:14:23 +0530'
category: update
#category: ['jekyll', 'guides', 'sample_category']
#thumbnail: /assets/img/posts/platformer2.gif
keywords: c#, csharp, monogame, 2d, game, development, video game
permalink: /blog/cross_platform_example_announcement/
usemathjax: true
---

As I’ve <a href="https://lajbert.github.io/blog/game_announcement/">mentioned a few months ago</a>, I’ve started working on a mobile game with <a href="https://github.com/Lajbert/MonolithEngine">my engine</a>. The first step of this is to prepare the engine to work on mobile, so I’ve been pretty busy with that lately. I’m using the <a href="https://lajbert.itch.io/platformer-demo">platformer game example</a> to test my game on mobile, and so far, after some refactors and fixes, it works really nicely. It was a bit challenging to figure out how to do these things with MonoGame and C#, but I’ve managed to get my head around it. The reason I’ve decided to use the platformer game example for testing is that this game utilizes basically everything I’ve implemented so far. I still have some fixes/changes ahead of me, but once they are done, I will merge the mobile changes into the master branch, and the platformer game example is going to showcase how to develop simultaneously to mobile and PC with my engine, from project setup to coding things.

However, I do have a feeling that nobody besides me will ever use this engine, so I will soon post a tutorial about how to develop for PC and mobile with MonoGame, starting from scratch, from project setup to running the same codebase on both platforms.

Update 2021.10.12: the source code for the platformer game for mobile and PC can be found under GameSamples\PlatformerGame on <a href="https://github.com/Lajbert/MonolithEngine">my Github repo</a>. The article about setting up the project structure for cross-platform development with MonoGame is <a href="https://lajbert.github.io/blog/cross_platform_project/">here</a>.