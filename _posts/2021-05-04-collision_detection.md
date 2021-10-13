---
layout: post
title:  "2 Fast 2 Curious: collision detection with fast moving objects"
summary: "Mimicking continuous collision detection in discrete time."
author: Lajbert
date: '2021-05-04 18:12:23 +0530'
category: tutorial
#category: ['jekyll', 'guides', 'sample_category']
#thumbnail: /assets/img/posts/platformer2.gif
keywords: c#, csharp, monogame, 2d, game, development, video game
permalink: /blog/collision_detection/
usemathjax: true
---

Finally, here it is: the second part of the movements and collisions tutorial series, this is where we will discuss, how to do collision checks inside the FixedUpdate. If you did not read <a href="https://lajbert.github.io/blog/fixed_timestep/">my article about fixed timesteps</a> and <a href="https://lajbert.github.io/blog/move_your_body/">movements</a>, read both of them now, because you’ll need them for this tutorial.
If you don’t have it, you can download the starter source code from <a href="https://drive.google.com/file/d/1RdGYdyJLR4CRtk4NvPeyEaD-a5WIKxmd/view?usp=sharing">here</a>, it’s the same where we left off the previous article about movements. You will need it in this tutorial.

This article was born based on <a href="https://deepnight.net/">Sébastien “deepnight” Benard</a>‘s (Dead Cells) amazing work, so I’d like to give him all the due credits.

Let’s quickly recap what we have at the moment: we have a FixedUpdate loop that is updated 30 times/second and we have a rectangle that we can move around with the arrow keys.  
What we will do in this tutorial is creating a simple, static (environmental) collision system, collide our rectangle with it and then see how we can guarantee collision detection, regardless of fast your object is moving (with certain conditions).

Although I will demonstrate the technique using static collisions, the solution I’m going to show here also works perfectly for dynamic collisions as well.

Let’s start at the beginning: what is a static collision? Static collision or environmental collision is a type of collision where a dynamic object (that can move around) is colliding with a static object (which never changes it’s position it the world space) and we detect and responds to that collision.  
You might want to ask now: why differentiating the collisions between static and dynamic objects? If you have a dynamic box collision system, can’t you use that for the environment? Yes, you can definitely use your dynamic collisions for the environment as well, but why would you want to do a more expensive dynamic box collision against objects that will never change it’s position in the world space? In most games, the ground, many platforms and obstacles are completely static, they will never move under any circumstance, so let’s have a very cheap, but effective collision check against those, and just do the more expensive dynamic collision checks against objects that can move like enemies, projectiles, etc. . Sounds legit, isn’t it?  
A very simple, yet effective implementation of this is called ‘single point collision’ (or one point collision), the idea is the following: you split up your level to a 2D grid and check whether there is a collider (or in this case, whether the 2D array has ‘true’ value in it) on the left, right, top and bottom of your object. If there is something, you stop the movement. This may seem very primitive at first, but even Dead Cells uses an advanced version of this kind of collisions. The implementation is similarly simple: a 2D boolean array can do the trick, the grid cells with true values are colliders, the rest aren’t.

Let’s decide about the collision grid size first, let’s pick a value that can demonstrate nicely what we are trying to show in this article, 16 pixels. Create a constant for it:

{% highlight csharp %}
readonly int GRID_SIZE = 16;
{% endhighlight %}

We will resize our window, so our test background will look nice:

{% highlight csharp %}
protected override void Initialize()
{
    base.Initialize();
    _graphics.PreferredBackBufferWidth = 512;
    _graphics.PreferredBackBufferHeight = 256;
    _graphics.ApplyChanges();
}
{% endhighlight %}

Let’s declare our variables we are going to use in the following:

{% highlight csharp %}
// collision grid
bool[,] grid = new bool[256, 512];
// random generator for random colors
private Random rnd = new Random();
// this will hold the rectangles to visualize collisions
Dictionary<Vector2, Texture2D> colliderVisualization = new Dictionary<Vector2, Texture2D>();
{% endhighlight %}

We will need a 2D grid that represents our colliders, but to make things more visible, we will also add rectangles with random colors where the colliders are, so it’s much easier to see what is happening on the screen. I will provide a very simple, easy to follow sample implementation, but my goal here is not to implement pixel perfect collisions. This implementation will not result in perfectly positioned colliders, but to demonstrate the idea, it’s good enough and easy enough for beginners to understand.  
Also, let’s change the size of our test rectangle to match our GRID size.

{% highlight csharp %}
protected override void LoadContent()
{
    _spriteBatch = new SpriteBatch(GraphicsDevice);
    rectangle = CreateRectangle(GRID_SIZE, Color.DarkGreen);
    // let's create a border on the screen from the colliders
    int x;
    for (int y = 0; y < 16; y ++)
    {
        for (x = 0; x < 32; x ++)
        {
            if (x == 0 || y == 0 || y == 15 || x == 31)
            {
                Color color = new Color();
                color.R = (byte)rnd.Next(256);
                color.G = (byte)rnd.Next(256);
                color.B = (byte)rnd.Next(256);
                color.A = 255;
                // create add the visualization of the colliders
                colliderVisualization.Add(new Vector2(x, y) * GRID_SIZE, CreateRectangle(GRID_SIZE, color));
                // register the collisions themselves in the 2D array
                grid[x, y] = true;
            }
        }
    }
    x = 15;
    for (int y = 1; y < 15; y++)
    {
        Color color = new Color();
        color.R = (byte)rnd.Next(256);
        color.G = (byte)rnd.Next(256);
        color.B = (byte)rnd.Next(256);
        color.A = 255;
        // create add the visualization of the colliders
        colliderVisualization.Add(new Vector2(x, y) * GRID_SIZE, CreateRectangle(GRID_SIZE, color));
        // register the collisions themselves in the 2D array
        grid[x, y] = true;
    }
}
{% endhighlight %}

And the basic collision detection code in the FixedUpdate:

{% highlight csharp %}
private void FixedUpdate()
{
    previousPosition = currentPosition;
    MoveObject(); 
    // calculate the grid coordinates of the object
    Vector2 gridCoordinates = new Vector2((int)currentPosition.X / GRID_SIZE, (int)currentPosition.Y / GRID_SIZE);
    if (gridCoordinates.X > 0 && gridCoordinates.X < 32 * GRID_SIZE && gridCoordinates.Y > 0 && gridCoordinates.Y < 16 * GRID_SIZE)
    {
        // if there is a collider on the left grid, stop the opject
        if (grid[(int)gridCoordinates.X - 1, (int)gridCoordinates.Y] && velocity.X < 0)
        {
            velocity.X = 0;
        }
        // if there is a collider on the right grid, stop the opject
        if (grid[(int)gridCoordinates.X + 1, (int)gridCoordinates.Y] && velocity.X > 0)
        {
            velocity.X = 0;
        }
        // if there is a collider on the top grid, stop the opject
        if (grid[(int)gridCoordinates.X, (int)gridCoordinates.Y - 1] && velocity.Y < 0)
        {
            velocity.Y = 0;
        }
        // if there is a collider on the bottom grid, stop the opject
        if (grid[(int)gridCoordinates.X, (int)gridCoordinates.Y + 1] && velocity.Y > 0)
        {
            velocity.Y = 0;
        }
    }
    currentPosition += velocity;
    velocity *= friction;
}
{% endhighlight %}

Don’t forget to update our Draw() method to actually draw those colliders 🙂

{% highlight csharp %}
protected override void Draw(GameTime gameTime)
{
    GraphicsDevice.Clear(Color.White);
    _spriteBatch.Begin();
    foreach (KeyValuePair<Vector2, Texture2D> entry in colliderVisualization)
    {
        _spriteBatch.Draw(entry.Value, entry.Key, Color.White);
    }
    Vector2 drawposition = Vector2.Lerp(previousPosition, currentPosition, ALPHA);
    _spriteBatch.Draw(rectangle, drawposition, Color.White);
    _spriteBatch.End();
    base.Draw(gameTime);
}
{% endhighlight %}

Now you should see something like this (although the colors of the colliders will be different for you):

<img src="https://lajbert.github.io/assets/img/posts/collisions1.gif" />

If you try your game now, you will notice that sometimes the collision works, but sometimes our rectangle just moves through the walls like they weren’t there. If it is not like that for you, just decrease the movementSpeed to slow down your object and see that the collision always happening, and then increase the speed to a much higher value and you’ll see the rectangle almost always going through the walls. I mentioned earlier, and you will also notice that the collision is not pixel perfect (sometimes the object has a little space between the collider and itself, other times there is an intersection between them), but let’s ignore this for now, that’s a topic for another time, I did not want to further complicate the code. The important bit here is that when a collision is detected, we can see it, because the object stops, but it doesn’t always happen when it should.  
This is due to the fact that collisions are being checked in the FixedUpdate loop, and collision detection is a discrete operation. This means that collisions can’t be continuously detected along a path, the detection will always happen in discrete steps for current position of the object. But this is updated only 30 times a second in the FixedUpdate loop, meaning that the rectangle can move more than the size of our collider and the collision is not detected, because the previous position was before the collider when it wasn’t colliding yet, but the next position is after the collider, when it isn’t colliding anymore.  
I made a terrible drawing in Paint to demonstrate the effect at the end of <a href="https://lajbert.github.io/blog/fixed_timestep/">my article about fixed timesteps</a>, you can head there to read about it and see the images describing the problem.

So how do we solve it? One obvious solution is to increase the rate of the FixedUpdate to let’s say 60 times/second. You can go ahead and try, it might solve our issue in this case, but if you increase the speed of the bullet (or have smaller colliders), you will notice that you are skipping collisions again. By increasing the frequency of the FixedUpdate, we do not actually solve the problem, only push it forward to encounter it later in a different situation. Well, maybe if you can guarantee that in your game, all objects will move relatively slowly and you won’t have too small colliders, you might be done here and increasing the fixed update frequency might have solved you problem (although the more frequent your FixedUpdate is, the more work you put on the CPU).

The real solution is changing the way we detect our collisions, for which we will have 2 criteria:

* We have to decide what is the smallest unit of collision we want to detect. At this point, the first thing to occur to your mind might be something like “Well, obviously I want to detect a collision even if the collider is just 1 pixel big!”, but believe me, chances are, you don’t need it. Hardly any video game needs collisions this accurate, and if you happen to be working on one, you might be way ahead of me and probably don’t need this article anyways (and I hope you will share your method with the game dev community!) 🙂 The smallest unit of your collision detection should be equal to your GRID_SIZE. A 16×16 grid is very common, fine enough for most games while the detection is still cheap. <a href="https://deepnight.net/tutorial/a-simple-platformer-engine-part-1-basics/">Dead Cells uses this grid size for collision detection</a>, and chances are, it will be good enough for you too. This means that we will guarantee that if a collider is at least 16×16 pixels, we will always detect it on the grid. Just so happens to be that this is the value we have chosen for our grid size when declaring it, so we’ve just saved ourselves 3 seconds to change that value! (which you’ve probably already wasted on reading how much time we saved).
* As I mentioned before, moving an object is a discrete operation and nothing can change this. But we can guarantee that one step in this discrete operation is never bigger than our chosen collider size, let’s say 16×16 and do extra detections when it’s needed. This means that it doesn’t matter how fast on object is moving, we will break down it’s movement to several smaller steps instead of 1 big step. Let’s say your object would move 1.5 times your grid size in one update, which would mean that we might have missed a collision. To prevent this, instead of moving 1.5 grid size in one step, we will move 0.75 of your grid size twice, which will guarantee that the collision will be detected, as long as the size of your collider is at least your grid size.

In short, we will break down our big movement step to several smaller steps and do extra collision detection for each step. With this method, we don’t have to universally increase our FixedUpdate timestep, the extra collision detection will only happen for fast moving objects, only when they are actually moving really fast, the rest of the objects are unaffected. If you have your character and some enemies walking on a map, and you shoot a rocket, the extra collision detection will happen only for the rocket and only when it’s moving faster than the threshold, the rest of the entities will not do extra collisions checks. Neat!

Let’s see what we are going to implement:

* First, we will calculate how big the current step is. If the value of ‘steps’ is 0, that means the object is at rest, if it’s 1 it means that object is moving with a maximum of one grid / update, if it’s bigger than 1 it means the object is moving faster than 1 grid in one update cycle.
* Then we break down the speed separately on the X and Y axis.
Take 2 examples:  
If the object is moving 12 pixels in one cycle (which is less than our 16 pixel grid), our ‘steps’ will be 1, and we will move all 12 pixels in one loop and do one collision detection.  
If the object is moving 18 pixels in one cycle (which is bigger than our 16 pixel grid), our ‘steps’ will be 2, and we will move 2×9 pixels instead, and do 2 collision detections.  

Let’s check the code, this is how the FixedUpdate looks like now:

{% highlight csharp %}
private void FixedUpdate()
{
    previousPosition = currentPosition;
    MoveObject();
    // total velocity
    float velocitySum = Math.Abs(velocity.X) + Math.Abs(velocity.Y);
    // how many steps needed for the current movement
    float steps = (float)(Math.Ceiling(velocitySum / GRID_SIZE));
    if (steps > 0)
    {
        // movement broken down to smaller steps if needed
        float stepX = velocity.X / steps;
        float stepY = velocity.Y / steps;
        while (steps > 0)
        {
            Vector2 gridCoordinates = new Vector2((int)currentPosition.X / GRID_SIZE, (int)currentPosition.Y / GRID_SIZE);
            // calculate the grid coordinates of the object
            // if there is a collider on the left grid, stop the opject
            if (grid[(int)gridCoordinates.X - 1, (int)gridCoordinates.Y] && velocity.X < 0)
            {
                velocity.X = 0;
                stepX = 0;
            }
            // if there is a collider on the right grid, stop the opject
            if (grid[(int)gridCoordinates.X + 1, (int)gridCoordinates.Y] && velocity.X > 0)
            {
                velocity.X = 0;
                stepX = 0;
            }
            currentPosition.X += stepX;
            // if there is a collider on the top grid, stop the opject
            if (grid[(int)gridCoordinates.X, (int)gridCoordinates.Y - 1] && velocity.Y < 0)
            {
                velocity.Y = 0;
                stepY = 0;
            }
            // if there is a collider on the bottom grid, stop the opject
            if ( grid[(int)gridCoordinates.X, (int)gridCoordinates.Y + 1] && velocity.Y > 0)
            {
                velocity.Y = 0;
                stepY = 0;
            }
            currentPosition.Y += stepY;
            // check dynamic collisions here if you have it implemented
            steps--;
        }
    }
    velocity *= friction;
}
{% endhighlight %}

Notice that we can even skip validating the array indexes at this point.  
Now, this code might not be the nicest or fastest implementation there is, but it’s easy to follow for a beginner and demonstrates the idea.  
If you try your game now, you can increase the speed of the rectangle to as high as you want, the box will never move through any of the walls:

<img src="https://lajbert.github.io/assets/img/posts/collisions1.gif" />

I’ve increased the movementSpeed to make it 100x faster than the original (from 10 to 1000, and 10 was already heavily skipping collisions); it’s actually so fast that I wasn’t even able to capture it properly. But you can increase it to as high as you want, the collisions will always be detected!  
And the best of all: this same technique works also on dynamic collisions! You just have to add your dynamic collision check to the FixedUpdate loop where we update the object’s position, and there you go! You will do the extra dynamic collision checks for fast moving objects, but for the rest, you will have as much as your FixedUpdate is configured for!

I hope this article was helpful and I was able to clearly explain what I wanted, but it you have any thoughts or questions, feel free to contact me!

If you had troubles following the code, you can download the complete source code from <a href="https://drive.google.com/file/d/1l86qdZ6vhFsc78SNKKbBuU2cQFfF-90u/view?usp=sharing">here</a>.

If you are interested in the implementation of the topics discussed on my blog, check out Monolith engine, my open source 2D game engine on <a href="https://github.com/Lajbert/MonolithEngine">GitHub</a>.

Stay tuned!