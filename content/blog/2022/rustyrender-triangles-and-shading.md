---
title: "RustyRender: Triangles and shading (lesson 2)"
author: Phineas Jensen
date: 2022-06-26
description: ""
tags:
  - rust
  - graphics
---

In the last lesson, we successfully [rendered a wire mesh of a model](/blog/2022/rustyrender-bresenhams-line-drawing-algorithm), which was something I was pretty excited about. I've never done that before! The next step in rendering these models correctly is drawing filled traingles, so our wire mesh (on the left) is filled out (on the right):

![diagonal line with spaces in it](/blog/2022/rustyrender/wire-and-filled.png)

By filling each triangle in the wire mesh with a shade of gray relative to its angle to the center of the image—we'll look at how that works towards the end—we have quite a good rendering of a person's head. You might notice the that the mouth, eyes, and ears all show some issues, which is one thing that's covered in the next lesson (so I'm told).

We ended last lesson with a fully functional line-drawing algorith, which can pretty easily be used to draw the outline of a triangle:

```rust
pub fn triangle<T: ColorSpace + Copy>(
    x0: isize,
    y0: isize,
    x1: isize,
    y1: isize,
    x2: isize,
    y2: isize,
    image: &mut Image<T>,
    color: T,
) {
  line(x0, y0, x1, y1, image, color);
  line(x1, y1, x2, y2, image, color);
  line(x2, y2, x0, y0, image, color);
}
```

![red triangle outline](/blog/2022/rustyrender/red-outline-triangle.png)

That's a pretty clunky function signature, with the x and y values listed individually, and so we're first going to take a detour and write a simple geometry library that will support 2- and 3-dimensional vectors and later allow us to implement some useful vector operations. The basic idea is pretty simple: 2-dimensional vectors will be structs with X and Y values, and 3-dimensional vectors will be structs with X, Y, and Z values.

We're gonna make these generic types, to allow us to use an integer type like `isize` for image coordinates and floating point numbers like `f64` for spatial coordinates (like those of the 3D model). We don't want the generic type to accept any old type for the coordinates (what exactly would a `Vec3<fs::File>` represent?), so we want to limit the acceptable types to numbers. Rust [doesn't have](https://stackoverflow.com/questions/37296351/is-there-any-trait-that-specifies-numeric-functionality) a built-in way to do that but there is the nice [num](https://crates.io/crates/num) crate built for that purpose, among other things. Adding that crate, we can build our 2-dimensional vector type, using the [Num](https://docs.rs/num/0.4.0/num/trait.Num.html) trait—which is implemented for all of the `u*`, `i*`, and `f*` types—in our trait bound:

```rust
use num::Num;

pub struct Vec2<T: Num> {
    pub x: T,
    pub y: T,
}

impl<T: Num> Vec2<T> {
    /// Build a vector with values at 0
    pub fn new() -> Self {
        Vec2 {
            x: T::zero(),
            y: T::zero(),
        }
    }

    /// Build a vector from two arguments
    pub fn from(x: T, y: T) -> Self {
        Vec2 { x, y }
    }

    // Build a vector from the first two indices of a slice
    pub fn from_slice(slice: &[T]) -> Self {
        Vec2 {
            x: slice[0],
            y: slice[1],
        }
    }
}
```

Our implementation for a 3-dimensional `Vec3` type is almost identical, so I've hidden it below to save space. Feel free to take a look:

{{% collapse "Vec3 implementation" %}}

```rust
pub struct Vec3<T: Num> {
    pub x: T,
    pub y: T,
    pub z: T,
}

impl<T: Num> Vec3<T> {
    pub fn new() -> Self {
        Vec3 {
            x: T::zero(),
            y: T::zero(),
            z: T::zero(),
        }
    }

    pub fn from(x: T, y: T, z: T) -> Self {
        Vec3 { x, y, z }
    }

    pub fn from_slice(slice: &[T]) -> Self {
        Vec3 {
            x: slice[0],
            y: slice[1],
            z: slice[2],
        }
    }
}
```
{{% /collapse %}}

Our triangle function is starting to look a bit better:

```rust
pub fn triangle<T: ColorSpace + Copy>(
    t0: Vec2<isize>,
    t1: Vec2<isize>,
    t2: Vec2<isize>,
    image: &mut Image<T>,
    color: T,
) {
    // ...
}
```

But now that we have some geometry types, we need to modify this function to draw _filled_ triangles, and that seems a little more difficult. Dmitry's original lesson outlines a few requirements for a good line-drawing algorithm and some hints at the basic outline:

> Traditionally a line sweeping is used:
> 
> 1. Sort vertices of the triangle by their y-coordinates;
> 2. Rasterize simultaneously the left and the right sides of the triangle;
> 3. Draw a horizontal line segment between the left and the right boundary points.

## My approach

So I got to work trying that out. As he says in his lessons, this is more difficult than it sounds. The first step, sorting by y-coordinates, is pretty easy: if our function takes t0, t1, and t2 as coordinate values, we just want to swap those so t0 is the left-most and t2 is the rightmost, which we can do by swapping each vertex with the other as necessary:

<span id="section-swap" />

```rust
pub fn triangle(
    t0: &Vec2<isize>,
    t1: &Vec2<isize>,
    t2: &Vec2<isize>,
    image: &mut Image<RGB>,
    color: RGB,
) {
    let mut t0 = t0;
    let mut t1 = t1;
    let mut t2 = t2;
    if t0.y > t1.y {
        (t0, t1) = (t1, t0);
    }
    if t0.y > t2.y {
        (t0, t2) = (t2, t0);
    }
    if t1.y > t2.y {
        (t1, t2) = (t2, t1);
    }
}
```

Having the y-coordinates in order makes it easier to figure out where to start and end each line. We know that the line stretching from `t0` to `t2` is going to have a longer y-length than any other line, so we can iterate over each y-coordinate that it spans and draw a line from where it falls on the x-axis to whatever line falls on the other side—and we can figure out which that line is based on the y-coordinate being iterated on at the moment. If the y-coordinate is less than the y-coordinate of `t1` (the middle point by y-coordinate), then the end point will fall on the `t0`–`t1` line. Otherwise, it'll fall on the `t1`–`t2` line. With that in mind, we can dig up some early algebra knowledge about lines to figure out what the relevant x-coordiantes are. Here are equations for a line:

```plain
y = bx + c      # (equation for a line, given the X-coordinate)
x = (y - c) / b # (equation for a line, given the Y-coordinate)
```

Where `b` is the _slope_ of the line, calculated as Y-distance per X-distance (e.g. if the line goes up two units for every one unit to the right, the slope is 2) and `c` is the intercept of the line, the Y-coordinate when X=0. Both of those values are easy to calculate: the slope is the distance between the Y-coordinates of two points divided by the distance between the X-coordinates, and the intercept is the Y-coordinate minus the X-coordinate times the slope (i.e. what the Y-coordinate would be if the intercept was 0). With those equations, I wrote some ugly but mostly functional code to implement the line-drawing algorithm:

```rust
// ...
    let a_slope = (t0.y - t2.y) as f32 / (t0.x - t2.x) as f32;
    let b_slope = (t0.y - t1.y) as f32 / (t0.x - t1.x) as f32;
    let a_intercept = t0.y as f32 - a_slope * t0.x as f32;
    let b_intercept = t0.y as f32 - b_slope * t0.x as f32;
    for y in t0.y..t1.y {
        line(
            &Vec2 {
                x: ((y as f32 - a_intercept) / a_slope) as isize,
                y,
            },
            &Vec2 {
                x: ((y as f32 - b_intercept) / b_slope) as isize,
                y,
            },
            image,
            color,
        );
    }
    let b_slope = (t1.y - t2.y) as f32 / (t1.x - t2.x) as f32;
    let b_intercept = t1.y as f32 - b_slope * t1.x as f32;
    for y in t1.y..t2.y {
        line(
            &Vec2 {
                x: ((y as f32 - a_intercept) / a_slope) as isize,
                y,
            },
            &Vec2 {
                x: ((y as f32 - b_intercept) / b_slope) as isize,
                y,
            },
            image,
            color,
        );
    }
// ...
```

A version of this with some modified variable names and diagnostic lines exists at commit [5a4c1a3](https://github.com/phinjensen/rustyrender/commit/5a4c1a3b178ac6d7744660a1ae5de52bd19ad653), and rendering a few test triangles we can see that it _mostly_ works:

![filled red triangles with green line showing some pixels that shouldn't be drawn](/blog/2022/rustyrender/red-filled-with-green-lines.png)

The issue, which may be difficult to see at a small size, is that there are some pixels to the left of the green line being rendered—probably due to differences in what pixels the line drawing algorithm renders and which ones are used as the end points for the line sweeping. Rather than try to debug this I decided to see how the tinyrenderer implementation works.

## A better line-sweeping approach

The beginning of tinyrenderer's better line-sweeping method is the same as my [earlier approach](#section-swap) where we start by sorting the vertices. After that, we find the total height and the partial height of the bottom half of the triangle, i.e. the portion from the lowest vertex to the height of the second vertex. Using these height numbers we can calculate the location at each point along the triangle's sides for each y-coordinate we're iterating over. By iterating from the bottom y-point to the top y-point, we calculate values `alpha` and `beta` to be the relative progress along the full and partial line segments, respectively. However, this time instead of calculating line functions ourselves, we can just use do calculations with the vertices we have instead of on their individual components. The original C++ code shows the following:

```c++
Vec2i A = t0 + (t2-t0)*alpha; 
Vec2i B = t0 + (t1-t0)*beta;
```

So we calculate the appropriate point on each line by taking the starting point `t0` and adding the vector from that point to the destination point _multiplied by our progress_. If you're unfamiliar with vector addition and multiplication, it might be worth reviewing some basic linear algebra and understand how that works. Linear algebra is not my strong area, so I'll instead recommend you look at some other resource[^1].

At any rate, our current Vec types don't support addition, subtraction, or multiplication by scalars, so we need to implement those. Luckily, Rust has nice traits that enable operator overloading in the [`Sub`](https://doc.rust-lang.org/std/ops/trait.Sub.html), [`Add`](https://doc.rust-lang.org/std/ops/trait.Add.html), and [`Mul`](https://doc.rust-lang.org/std/ops/trait.Mul.html) traits. Subtraction and Addition are pretty straightforward:

```rust
impl<T: Num + Copy> Sub for Vec2<T> {
    type Output = Self;

    fn sub(self, rhs: Self) -> Self::Output {
        Vec2 {
            x: self.x - rhs.x,
            y: self.y - rhs.y,
        }
    }
}

impl<T: Num + Copy> Add for Vec2<T> {
    type Output = Self;

    fn add(self, rhs: Self) -> Self::Output {
        Vec2 {
            x: self.x + rhs.x,
            y: self.y + rhs.y,
        }
    }
}
```

These traits both allow a `Rhs` paramter, which by default is `Self` and that's exactly what we want. We're adding the ability to add and subtract two vectors together. The output of that operation is also a vector, so the `Output` type of the trait is also `Self`. Within the function bodies we just create and return a new `Vec2` with the x and y coordinates being the difference or addition of the argument's coordinates. Not too bad.

For our function we want to be able to multiply by a scalar, and for that we just need to implement `Mul` with the `Rhs` set to `f32` (the float type we're using for our `alpha` and `beta` values):

```rust
impl Mul<f32> for Vec2<isize> {
    type Output = Self;

    fn mul(self, rhs: f32) -> Self::Output {
        Vec2 {
            x: (self.x as f32 * rhs) as isize,
            y: (self.y as f32 * rhs) as isize,
        }
    }
}
```

With those traits implemented, we can now add and subtract vectors and multiply them by f32 values, making our bottom-half drawing code look pretty clean:

```rust
pub fn triangle(...) {
    // ...
    let height = (t2.y - t0.y) as f32;
    let segment_height = (t1.y - t0.y) as f32;
    for y in t0.y..t1.y {
        let alpha = (y - t0.y) as f32 / height;
        let beta = (y - t0.y) as f32 / segment_height;
        let mut a = t0 + (t2 - t0) * alpha;
        let mut b = t0 + (t1 - t0) * beta;
        if a.x > b.x {
            (a, b) = (b, a);
        }
        for x in a.x..=b.x {
            image.set(x as usize, y as usize, color).unwrap();
        }
    }
}
```

After calculating our new vectors representing the points on each side of the line we swap them if necessary so the `a.x < b.x` and then we can iterate over every point from the left to the right side, drawing the color at that point.

Drawing the top half is almost the same:

```rust
// ...
    let segment_height = (t2.y - t1.y) as f32;
    for y in t1.y..=t2.y {
        let alpha = (y - t0.y) as f32 / height;
        let beta = (y - t1.y) as f32 / segment_height;
        let mut a = t0 + (t2 - t0) * alpha;
        let mut b = t1 + (t2 - t1) * beta;
        if a.x > b.x {
            (a, b) = (b, a);
        }
        for x in a.x..=b.x {
            image.set(x as usize, y as usize, color).unwrap();
        }
    }
// ...
```

We calculate the new segment height and iterate from the midpoint y-coordinate to the last y-coordinate, calculate our alpha and beta values again, and do the same kind of vector math to get the appropriate points on each line. With that, we can now draw some nice triangles:

![fully-drawn trinagles in red, white and green](/blog/2022/rustyrender/full-triangles.png)

The functioning source code for this method is available at commit [`9a17e504e15bf404547445acb5b286d7548524dd`](https://github.com/phinjensen/rustyrender/tree/9a17e504e15bf404547445acb5b286d7548524dd).

## A more modern solution



[^1]: I haven't used their linear algebra videos myself, but [Khan Academy](https://www.khanacademy.org/math/linear-algebra) hasn't let me down in the past.
