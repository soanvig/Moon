---
layout: post
title: How percentile background-position works?
tags: [CSS]
comments: true
excerpt: This post explains how percentile background-position works, and how to calculate its true position.
---

(This post is in English language to reach the widest community)

## Background-position

`background-position` is [CSS rule](https://developer.mozilla.org/en-US/docs/Web/CSS/background-position?v=control) which is used to position `background-image` inside *background-area* (the area of container).

About this rule see [its reference](https://developer.mozilla.org/en-US/docs/Web/CSS/background-position?v=control).

## The theory

In theorem the `background-position` written as % calculates the position as follows:

1. Calculate `[given value]% * [background-image size]` - this is reference point on image
2. Place reference point (and image, which is bound to it) at `[given value]% * [background area size]`

Because of this approach image of 100x100px with `background-position: 50% 50%` is placed in perfect center of background area. It's reference point is moved from typical `left-top` to `50%-50%` of itself, then this reference point is placed in the `50%-50%` of background area.

This way using `background-position` set to (X-axis):

1. `100% 0` places right edge of image on the right edge of container
2. `0% 0` places left edge of image on the left edge of container
3. `50% 0` places middle point of image in the middle of container

## The problem

But what happens, when we want to put an image in `20%` on X-axis (so it's left edge is exactly in the 20% of the container). It won't work, because the 20% value will move image's reference point.

I've came across this problem, when I tried to create `linear-gradient` (which is background image) of width 50% of container, placed 20% from left edge of container.

## The solution

To solve this problem, we need to use math. I'll present described problem on the image below:

![]({{ site.url }}/assets/images/how-background-position-works/scheme.jpg)

Where:

- the black box = container
- the red box = background image
- x = the distance between real reference point and left edge of container - the given value
- y = the size of background image (set by `background-size`)
- z = the distance between left edge of container and left edge of image

Let's start with some initial CSS code:

```css
.container {
    background-image: linear-gradient(red, red); /* this will create solid image */
    background-size: y% 100%; /* vertically filled container */
    background-position: x% 0%;
}
```

As we know, the z distance can be calculated as follows (it comes from the scheme above):

![z = x - x * y]({{ site.url }}/assets/images/how-background-position-works/calc1.png)

The `x * y` is the moved image's reference point.

We now `z` we want to get (20%), and we know `y` (50%). The `x` is the search value. We can proceed with transformations:

![z = x * (1 - y); x = z / (1 - y)]({{ site.url }}/assets/images/how-background-position-works/calc2.png)

Using this equation for `x` we can calculate the value we must give to `background-position` to get the distance between container's border and image's edge (treat 1 as 100%):

![x = 20% / (100% - 50%) = 40%]({{ site.url }}/assets/images/how-background-position-works/calc3.png)

And the `40%` is the value which interests us:

```css
.container {
    background-image: linear-gradient(red, red);
    background-size: 50% 100%;
    background-position: 40% 0%;
}
```
