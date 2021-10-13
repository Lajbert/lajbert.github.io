---
layout: post
title:  "Fix your timestep - in MonoGame"
summary: "Advantages and example implementation of fixed timestep in MonoGame"
author: Lajbert
date: '2021-05-02 18:12:23 +0530'
category: tutorial
#category: ['jekyll', 'guides', 'sample_category']
#thumbnail: /assets/img/posts/platformer2.gif
keywords: c#, csharp, monogame, 2d, game, development, video game
permalink: /blog/fixed_timestep/
usemathjax: true
---

There is a famous article circling the internet since more than 15 years now: <a href="https://gafferongames.com/post/fix_your_timestep/">Glenn Fiedler’s Fix your Timestep!</a>, which is basically the 101 of fixed timesteps explanations. However, it might difficult for a beginner to understand and implement it, so I’ve decided to provide a beginner friendly starter implementation and some use cases. If you haven’t read the original article, do it now before you carry on with this post.

First, let’s recap what fixed timestep is why is it sooooo good.  
The game loop of MonoGame is the following: Update() and Draw() calls follow each other as long as your game is running – before every Draw() call, there is always an Update() call. Typically, we do game logic related code inside the Update method (collisions, AI, etc.) and we do every render related code in the Draw method, keeping it as simple and minimal as possible. This means that if your game runs with 30 FPS, the Update is called 30 times in each second (before the Draw calls). If your game is running with 600 FPS, the Update is called 600 times a second! See the problem now?

* Optimizing the code for variable framerate can be tricky, especially physics can suffer the ‘elapsed time’ difference between 30 FPS and 1000+ FPS (we’re talking about 2D games here, 1000+ FPS is not uncommon for modern GPUs)
* Hammering the CPU is bad. Why would you want to do 1000+ collisions / second? Let’s take this one step further: Why would you even want to have more than 30 collision checks / second? First, it’s useless because the user won’t notice it anyways (if your implementation is correct). Second, on mobile/handheld devices, hammering the CPU will drain the battery really fast, and it’s kind of hard to defend why a Super Mario-like game eats up the battery and heats up the phone 🙂
* You can guarantee predictability with fixed time steps: you will know that certain events will take place exactly the way you want it, always, regardless of how fast the game runs on the user’s machine.

The solution for the above mentioned problems: fixed timesteps! This means that next to your regular Update() call, we will implement a FixedUpdate() call, which runs with a pre-configured frequency: 30 times/second is most typical setting. **And the best of all: the rendering is not affected in any way**, so even though your game logic runs only 30 times a second, **the player will still see butter smooth image**, as high framerate as their rig can produce! Isn’t it cool?

Key differences between Update and the soon-to-be-implemented FixedUpdate():

* Update() is called as much as your game framerate, while FixedUpdate is called with a fixed frequency. We will configure it to run 30 times/second, regardless of your game’s framerate.
* Update() will have the actual elapsedTime passed, because it’s needed for accurate game logic. FixedUpdate has no such value, the elapsedTime will always be 1 and you should multiply with it. You should think about it as one unit of “run” instead of a time period. If you want to double your FixedUpdate timestep for any reason during the game, passing 0.5 as elapsedTime for the FixedUpdate() guarantees that you will see things exactly the same way as 30 FPS and 1 elapsedTime values, but with 2x as much FixedUpdate() calls. Or, if you don’t change your fixed timestep frequency and just decrease this value, you can have a very cool bullet time effect!

If you <a href="https://docs.monogame.net/articles/getting_started/0_getting_started.html">create an empty MonoGame project</a>, you can follow along the code.  
Let’s add some code, I’m using the rectangle drawing method from <a href="https://lajbert.github.io/blog/monogame_2d_primitives/">my earlier post about drawing primities</a> to display the example objects on the screen.

{% highlight csharp %}
public Texture2D CreateRectangle(int size, Color color)
{
    Texture2D rect = new Texture2D(_graphics.GraphicsDevice, size, size);
    Color[] data = new Color[size * size];
    for (int i = 0; i < data.Length; ++i) data[i] = color;
    rect.SetData(data);
    return rect;
}
{% endhighlight %}

Let’s declare the class variables we are going to use at the top of our class. We are going to use these rectangles to demonstrate the logic.

{% highlight csharp %}
// we are updating the first rectangle in the normal Update() loop
private Texture2D normalObject;
private Vector2 normalObjectPos = new Vector2(30, 30);
private float normalObjectSpeed = 0.2f;
 
// we are updating the second rectangle in the FixedUpdate() loop
private Texture2D fixedUpdateObject;
private Vector2 fixedUpdateObjectPos = new Vector2(30, 100);
private float fixedUpdateObjectSpeed = 6.6f;
 
// this will be the FixedUpdate frequency, we set it to 30 FPS
private float fixedUpdateDelta = (int)(1000 / (float)30);
 
// helper variables for the fixed update
private float previousT = 0;
private float accumulator = 0.0f;
private float maxFrameTime = 250;
 
// this value stores how far we are in the current frame. For example, when the 
// value of ALPHA is 0.5, it means we are halfway between the last frame and the 
// next upcoming frame.
private float ALPHA = 0;
{% endhighlight %}

Now we are going to initialize these rectangles in the LoadContent() method:

{% highlight csharp %}
protected override void LoadContent()
{
    _spriteBatch = new SpriteBatch(GraphicsDevice);
 
    normalObject = CreateRectangle(64, Color.Red);
    fixedUpdateObject = CreateRectangle(64, Color.Blue);
}
{% endhighlight %}

Now let’s create our FixedUpdate method with the code to update the position of the fixedUpdateObject:

{% highlight csharp %}
private void FixedUpdate()
{
    if (fixedUpdateObjectPos.X < 30 || fixedUpdateObjectPos.X > 600)
    {
        fixedUpdateObjectSpeed *= -1;
    }
 
    fixedUpdateObjectPos.X += fixedUpdateObjectSpeed;
}
{% endhighlight %}

Now, we have to change the Update() method to implement the fixed update. The idea is the following (although this is explained in Glenn Fiedler’s Fix your Timestep! post): we are going to measure the elapsed time of the game in an accumulator. While accumulator is bigger than our desired fixed update delay, we are going to run one FixedUpdate and decrease the accumulator by the fixed update delay. We also keep track of our progress in the value called ALPHA, this shows how far we are in the current frame. For example, ALPHA value 0.5 means that we are halfway between the previous frame and the next upcoming frame. We will need this variable later.  
Here is the code:

{% highlight csharp %}
protected override void Update(GameTime gameTime)
{
    if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed || Keyboard.GetState().IsKeyDown(Keys.Escape))
        Exit();
 
 
    if (previousT == 0)
    {
        previousT = (float)gameTime.TotalGameTime.TotalMilliseconds;
    }
 
    float now = (float)gameTime.TotalGameTime.TotalMilliseconds;
    float frameTime = now - previousT;
    if (frameTime > maxFrameTime)
    {
        frameTime = maxFrameTime;
    }
         
    previousT = now;
 
    accumulator += frameTime;
 
    while (accumulator >= fixedUpdateDelta)
    {
        FixedUpdate();
        accumulator -= fixedUpdateDelta;
    }
 
    // this value stores how far we are in the current frame. For example, when the 
    // value of ALPHA is 0.5, it means we are halfway between the last frame and the 
    // next upcoming frame.
    ALPHA = (accumulator / fixedUpdateDelta);
 
    // let's update the normal rectangle position in this Update loop
    if (normalObjectPos.X < 30 || normalObjectPos.X > 600)
    {
        normalObjectSpeed *= -1;
    }
 
    normalObjectPos.X += normalObjectSpeed * (float)gameTime.ElapsedGameTime.TotalMilliseconds;
 
    base.Update(gameTime);
}
{% endhighlight %}

Now, let’s modify the Draw method to display our rectangles:

{% highlight csharp %}
protected override void Draw(GameTime gameTime)
{
    GraphicsDevice.Clear(Color.White);
 
    // TODO: Add your drawing code here
 
    _spriteBatch.Begin();
    _spriteBatch.Draw(normalObject, normalObjectPos, Color.White);
    _spriteBatch.Draw(fixedUpdateObject, fixedUpdateObjectPos, Color.White);
    _spriteBatch.End();
 
    base.Draw(gameTime);
}
{% endhighlight %}

Let’s run our program and inspect the result:

<img src="https://lajbert.github.io/assets/img/posts/fixed_update1.gif" />