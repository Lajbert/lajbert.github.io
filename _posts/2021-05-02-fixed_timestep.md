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

Hmm… something seems off. The .gif might now show it that well, but if you run the program, you’ll notice that while the red square has butter smooth movement, the blue one, which we update in the FixedUpdate() method, isn’t so smooth. Almost as if it was running with only 30 FPS… Because it is! Well, kind of…  
Let’s say your program is running with 120 FPS, and the fixed update is set to 30 FPS. This means that while the position of the red square is update with each frame (120 times a second), the position of blue frame is only updated at every 30 frames, meaning that all 4 frames will display the blue square on the exact same position before the FixedUpdate kicks in and updates it again!  
But wait. So was I lying when I told you that the users will see smooth gameplay with only 30 fixed update/second in the background? Nope, I wasn’t! The solution is: **interpolation**!

Remember I told you about the ALPHA variable before? This is time when we need it!  
Let’s say we just had a FixedUpdate call which updated the blue square’s X position to 50, and next FixedUpdate (whenever it comes) will update it to 60.  
What does it mean now? Currently, the X coordinate is 50, so the game loop will look like this:  
FixedUpdate() //setting the X coordinate to 50  
Draw() // drawing the blue square at X coordinate 50  
Draw() // drawing the blue square at X coordinate 50  
Draw() // drawing the blue square at X coordinate 50  
Draw() // drawing the blue square at X coordinate 50  
FixedUpdate() //setting the X coordinate to 60  
Draw() // drawing the blue square at X coordinate 60  
Draw() // drawing the blue square at X coordinate 60  
Draw() // drawing the blue square at X coordinate 60  
Draw() // drawing the blue square at X coordinate 60  

Wouldn’t it be cool, if, instead of drawing the square in the same position 4 times, we would somehow calculate where it should be in the current Draw call, and draw it there? Something like this:

FixedUpdate() //setting the X coordinate to 50, the previous position is 40  
Draw() // drawing the blue square at X coordinate 42  
Draw() // drawing the blue square at X coordinate 44  
Draw() // drawing the blue square at X coordinate 46  
Draw() // drawing the blue square at X coordinate 48  
FixedUpdate() //setting the X coordinate to 60, the previous position is 50  
Draw() // drawing the blue square at X coordinate 50  
Draw() // drawing the blue square at X coordinate 52  
Draw() // drawing the blue square at X coordinate 54  
Draw() // drawing the blue square at X coordinate 58  
FixedUpdate() //setting the X coordinate to 70, the previous position is 60  
Draw() // drawing the blue square at X coordinate 60  
Draw() // drawing the blue square at X coordinate 62  
and so on…

See how much smoother would this be? This where <a href="https://en.wikipedia.org/wiki/Linear_interpolation">linear interpolation</a> will help us!  
Let’s create a third square, but for this one, we will save the previous position and interpolate between the previous and the current position using the ALPHA value I explained earlier.

Let’s add a third class variable:

{% highlight csharp %}
// we are updating the third rectangle in the FixedUpdate() loop
// but we also interpolate it's position
Texture2D lerpObject;
Vector2 lerpObjectPos = new Vector2(30, 170);
Vector2 lerpObjectPrevPos = Vector2.Zero;
private float lerpObjectSpeed = 6.6f;
{% endhighlight %}

And initialize it:

{% highlight csharp %}
protected override void LoadContent()
{
    _spriteBatch = new SpriteBatch(GraphicsDevice);
 
    normalObject = CreateRectangle(64, Color.Red);
    fixedUpdateObject = CreateRectangle(64, Color.Blue);
    lerpObject = CreateRectangle(64, Color.DarkGreen);
}
{% endhighlight %}

And now, let’s do a simple <a href="https://en.wikipedia.org/wiki/Linear_interpolation">linear interpolation</a> between the old position and the new one. Let’s modify the FixedUpdate method to store the third rectangle’s previous position and update it’s current position:

{% highlight csharp %}
private void FixedUpdate()
{
    if (fixedUpdateObjectPos.X < 30 || fixedUpdateObjectPos.X > 600)
    {
        fixedUpdateObjectSpeed *= -1;
    }
 
    fixedUpdateObjectPos.X += fixedUpdateObjectSpeed;
 
    // saving the previous position of the green rectangle
    lerpObjectPrevPos = lerpObjectPos;
 
    if (lerpObjectPos.X < 30 || lerpObjectPos.X > 600)
    {
        lerpObjectSpeed *= -1;
    }
 
    // setting the current position of the green rectangle
    lerpObjectPos.X += lerpObjectSpeed;
}
{% endhighlight %}

Now, let’s modify the Draw method to interpolate between the previous position and the current one:

{% highlight csharp %}
protected override void Draw(GameTime gameTime)
{
    GraphicsDevice.Clear(Color.White);
 
    // TODO: Add your drawing code here
 
    _spriteBatch.Begin();
    _spriteBatch.Draw(normalObject, normalObjectPos, Color.White);
    _spriteBatch.Draw(fixedUpdateObject, fixedUpdateObjectPos, Color.White);
 
    // we'll save the in-between interpolated positions in drawPosition 
    // and draw the object there instead of it's current position
    Vector2 drawPosition = Vector2.Lerp(lerpObjectPrevPos, lerpObjectPos, ALPHA);
    _spriteBatch.Draw(lerpObject, drawPosition, Color.White);
    _spriteBatch.End();
 
    base.Draw(gameTime);
}
{% endhighlight %}

Now, let’s run our program:

<img src="https://lajbert.github.io/assets/img/posts/fixed_update2.gif" />

It might not be that visible on the .gif, but if you run the program, you’ll see how smooth the green square is, just like the red one, although it’s position is updated only in the FixedUpdate method!
Now you might be wondering: Why go through all this hassle when I can just update the position simply in the Update() loop? Because most of the time, moving an object in the game is much more than just simply adding to it’s X or Y coordinates: in real life, it’s the result of a bunch of expensive calculations: collisions (maybe multiple different collision types), physics updates, AI, forces, objects interacting with each other, and so on… These are pretty expensive things, which you really want do in a FixedUpdate loop and compared to them, a linear interpolation is almost free. Doing these CPU heavy updates only 30 times a second and determining the in-between positions with linear interpolation for the rest of the frames is much easier on the hardware, and for the reasons explained in the beginning of the article, is the way to go.

You might still have some questions in your head: what about the collisions? If your collision check is running in the FixedUpdate, is 30 check/second good enough? What if I have a bullet that is moving fast, and this happens:

<img src="https://lajbert.github.io/assets/img/posts/fixed_update3.png" />

The hollow gray squares shows the past positions of the bullet, the black square is the current one, the red line is a wall. The FixedUpdate frequency was not high enough to detect this collision with the red line, because the positions are only updated 30 times/second, and the bullet is moving a lot during one cycle, going through the wall, which is bad. The obvious solution is increasing the FixedUpdate frequency. Let’s say we did it, and this is what we got:

<img src="https://lajbert.github.io/assets/img/posts/fixed_update4.png" />

We have increased the FixedUpdate frequency, but the bullet is moving really fast and movement is still not frequent enough to detect that collision. We could further increase the FixedUpdate frequency and it might solve the problem in the current scenario, but what happens if there is another object that moves even faster, or you have thinner collider? Increase the frequency further? Until when? Or should we calculate collisions for the in-between interpolated position? That would just defeat the purpose of the whole FixedUpdate, as we would run the collision at every frame that the computer renders and not in our FixedUpdate loop, and we would be back at where we started!  
Is there anything we can do to detect every collision, while still running the collision checks in the FixedUpdate configured to 30 FPS, regardless of how fast an object is moving? Yes, there is a way to do it (with certain conditions), we will look at this in the <a href="https://lajbert.github.io/blog/move_your_body/">next article series</a> about movement and collision basics!

If you got lost in the coding, you can download the complete project files from <a href="https://drive.google.com/file/d/1cWr2fGAV8KJpsS9VrZ8Ow5ckRxGJQUi_/view?usp=sharing">here</a>. It was tested with MonoGame 3.8.

If you are interested in the implementation of the topics discussed on my blog, check out Monolith engine, my open source 2D game engine on <a href="https://github.com/Lajbert/MonolithEngine">GitHub</a>.

Stay tuned!