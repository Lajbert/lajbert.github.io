---
layout: post
title:  "Drawing 2D primitivies in MonoGame"
summary: "While working on new features and prototypes, it always makes my life much easier if I can visualize what is happening on the screen, for example: visualizing the result of a raycast or the pivot of a sprite."
#author: Lajbert
date: '2021-01-14 18:00:23 +0530'
category: tutorial
#category: ['jekyll', 'guides', 'sample_category']
thumbnail: /assets/img/posts/primitive.png
keywords: c#, csharp, monogame, 2d, game, development, video game
permalink: /blog/monogame_2d_primitives/
usemathjax: true
---


While working on new features and prototypes, it always makes my life much easier if I can visualize what is happening on the screen, for example: visualizing the result of a raycast or the pivot of a sprite. For that, I’m using primitives with the following simple code snippets:

{% highlight csharp %}
// rectangles
public static Texture2D CreateRectangle(int size, Color color)
{
    Texture2D rect = new Texture2D(GraphicsDeviceManager.GraphicsDevice, size, size);
    Color[] data = new Color[size * size];
    for (int i = 0; i < data.Length; ++i) 
    {
        data[i] = color;
    }
    rect.SetData(data);
    return rect;
}
{% endhighlight %}

{% highlight csharp %}
//drawing circles
public static Texture2D CreateCircle(int diameter, Color color)
{
    Texture2D texture = new Texture2D(GraphicsDeviceManager.GraphicsDevice, diameter, diameter);
    Color[] colorData = new Color[diameter * diameter];

    float radius = diameter / 2f;
    float radiusSquared = radius * radius;

    for (int x = 0; x < diameter; x++)
    {
        for (int y = 0; y < diameter; y++)
        {
            int index = x * diameter + y;
            Vector2 pos = new Vector2(x - radius, y - radius);
            if (pos.LengthSquared() <= radiusSquared)
            {
                colorData[index] = color;
            }
            else
            {
                colorData[index] = Color.Transparent;
            }
        }
    }

    texture.SetData(colorData);
    return texture;
}
{% endhighlight %}

Drawing a line in MonoGame is a bit trickier, I was not able to find an easy way to create a Texture2D for it, so I’m just going to share the code I’m using to render a line:
{% highlight csharp %}
public class Line
{
    private Vector2 Origin;
    private Vector2 Scale;

    public Vector2 From;
    public Vector2 To;

    private Vector2 fromSaved;
    private Vector2 toSaved;

    private Color color;
    private float thickness;

    private float angleRad;
    private float length;

    public Line(Vector2 from, Vector2 to, Color color, float thickness = 1f)
    {
        this.From = fromSaved = from;
        this.To = toSaved = to;
        this.thickness = thickness;
        this.color = color;
        Sprite = CreateRectangle(1, Color.White);
        length = Vector2.Distance(from, to);
        angleRad = AngleFromVectors(from, to);
        Origin = new Vector2(0f, 0f);
        Scale = new Vector2(length, thickness);
    }

    public Line(Vector2 from, float angleRad, float length, Color color, float thickness = 1f)
    {
        this.From = fromSaved = from;
        this.thickness = thickness;
        this.color = color;
        // see the function about creating rectangle
        Sprite = CreateRectangle(1, Color.White);
        this.length = length;
        this.angleRad = angleRad;
        To = toSaved = EndPointOfLine(from, length, this.angleRad);
        Origin = new Vector2(0f, 0f);
        Scale = new Vector2(length, thickness);
    }

    // call this if you want to dynamically change where the line ends, for example to see an intersection of a raycast result
    public void SetEnd(Vector2 end)
    {
        To = end;
        length = Vector2.Distance(From, To);
        angleRad = AngleFromVectors(From, To);
        Scale = new Vector2(length, thickness);
    }

    public void Reset()
    {
        From = fromSaved;
        To = toSaved;
        length = Vector2.Distance(From, To);
        angleRad = AngleFromVectors(From, To);
        Scale = new Vector2(length, thickness);
    }

    public static Vector2 EndPointOfLine(Vector2 start, float length, float angleRad)
    {
        return new Vector2(start.X + length * (float)Math.Cos(angleRad), start.Y + length * (float)Math.Sin(angleRad));
    }

    public static float AngleFromVectors(Vector2 v1, Vector2 v2)
    {
        return (float)Math.Atan2(v2.Y - v1.Y, v2.X - v1.X);
    }
}
{% endhighlight %}