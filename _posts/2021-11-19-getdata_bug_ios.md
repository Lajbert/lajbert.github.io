---
layout: post
title:  "MonoGame iOS UI freeze"
summary: "But read this even if you are not developing to iOS"
author: Lajbert
date: '2021-11-19 12:19:23 +0530'
category: tutorial
#category: ['jekyll', 'guides', 'sample_category']
#thumbnail: /assets/img/posts/platformer2.gif
keywords: c#, csharp, monogame, 2d, game, development, video game
permalink: /blog/get_data_ios/
usemathjax: true
---

Appearently, MonoGame 3.8.0.1641 has a bug in the Texture2D.GetData() call on iOS: if you call this method from either Update or Draw methods, the UI thread on the phone freeze (although the game will be running in the background). The solution is simple (at least it was for me): move all your GetData calls out of the Draw and Update methods to the game's initialization part.
Chances are you are using GetData to alter a texture. If this is the case, you can probably replace all your GetData calls with using <a href="http://rbwhitaker.wikidot.com/render-to-texture">RenderTarget2D</a> which is not using GetData at all.
If you are using GetData to analyse a texture, you can probably just move it to LoadContent() method at the beginning.
And there is a second reason why should get rid of your GetData calls.

This part is appicable for everyone, not just for those who are targeting iOS: GetData alwyas reads the whole texture as of MonoGame 3.8.0.1641, which has a huge impact on your game's performance. So even if you are you unaffected by the iOS bug, its a good idea to do all these calls during initialization, otherwise it can (and in case of bigger textures, it will) decrease your game's performance, which is very precious, especially on mobile platforms.