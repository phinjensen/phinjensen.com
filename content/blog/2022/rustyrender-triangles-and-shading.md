---
title: "RustyRender: Triangles and shading (lesson 2)"
author: Phineas Jensen
date: 2022-06-26
description: ""
tags:
  - rust
  - graphics
---

In the last lesson, we successfully [rendered a wire mesh of a model](/blog/2022/rustyrender-bresenhams-line-drawing-algorithm), which was something I was pretty excited about. I've never done that before! Now, as someone who has absorbed some graphics knowledge through osmosis (playing games, hearing tech announcements, light reading, etc.) I know that our next step needs to be rendering filled triangles because 3D rendering is all based on triangles. That's actually something I had never wondered about until writing this post, but there's actually some [pretty interesting reasons](https://stackoverflow.com/a/6100615/7355242) that triangles are important in 3D graphics.

Drawing a triangle should be pretty easy, right? 


---

- https://www.quora.com/Why-does-graphics-hardware-only-render-triangles?share=1
- https://stackoverflow.com/questions/6100528/why-do-3d-engines-primarily-use-triangles-to-draw-surfaces

