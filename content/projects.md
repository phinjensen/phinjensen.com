I don't have a ton of free time these days, but I try to use that free time productively. When I'm successful, these are the kinds of things I work/have worked on.

(work in progress)

## Technical

### rlox — an implementation of Robert Nystrom's Lox in Rust

February 2025–present

### Koja — private location sharing for Android

October 2024–present

### VajehSabt — background listening Persian translation

March–April 2024

### SeeYouLater — CLI and browser bookmarking tool

July 2022–November 2023

### BYU open room finder — tool for finding unused classrooms on BYU campus

February 2022-April 2023

### [raytracer — software raytracer written in Rust](https://github.com/phinjensen/raytracer)

![colored balls and triangles with a completely reflective ball at the bottom and harsh light coming from the right](raytracer.jpg)

One project for CS 455: Computer Graphics at BYU was to build a raytracer from scratch. They recommended C++ because the renders could get pretty slow and that's what the TAs were familiar with, but I thought it'd be a good chance to use Rust, and I had a ton of fun with it. We had to output images in the PPM format, support spheres and triangles with diffuse and reflective textures, and allow each ray to bounce at least three times. An interesting mistake I made at one point was incorrectly writing the image size in the output. I didn't notice the mistake because the GIMP happily ignored that metadata and rendered the image without any problem, but the grader used an image viewer that wasn't so loose with the format. I was able to resubmit without any points lost 😁.

This would be a fun project to continue. Things I could add:

- transparent objects
- parallel processing

February–March 2023

### [Ultra Geo Master — a web-based geography game skeleton](https://github.com/phinjensen/ultra-geo-master)

Growing up, my family would play the board game _[Where in the World?](https://boardgamegeek.com/boardgame/9646/where-in-the-world)_—specifically a version where we would take turns drawing a card with various information about a country for another person, saying the name of the country on the card, and having the other person guess where it is on the map. Each country was worth points based on its area; e.g. Ukraine was worth 1 point (being the largest country in Europe, excluding Russia, which was included in the Asia map/deck) while Vatican City was worth 45.

In this app, a simple backend server provides card and map data, so players can start a game with a map on a shared screen (such as a TV or laptop) and "join" the game on a phone, where they are given a deck of cards. The state of the deck was shared between each player in a game, so if you drew Germany, no one else in your game would draw it. The frontend was built with React.

The most fun and interesting part of this project was building slightly interactive SVG maps for display on the shared screen. I built the maps using shapefiles from [Natural Earth](https://www.naturalearthdata.com/), which is a fantastic project. I imported them into QGIS, set up some styles (including labels with the countries numbered by population, rather than area), and even did some trickery to include enough data for JavaScript to be able to identify what country is being hovered by the mouse.

December 2022

### [rustyrender — an implementation of tinyrenderer in Rust](https://github.com/phinjensen/rustyrender)

This is a partial implementation of Dmitry Sokolov's [tinyrenderer](https://github.com/ssloy/tinyrenderer/) in Rust. It's the first significant thing I wrote in Rust, and in order to learn the language and 3D rendering better, I wrote a series of blog posts to go along with each lesson:

- [Lesson 0: Setup](http://localhost:1313/blog/2022/rustyrender-tinyrender-in-rust-lesson-0/)
- [Lesson 1: Bresenham’s line-drawing algorithm](http://localhost:1313/blog/2022/rustyrender-tinyrender-in-rust-lesson-0/)

Okay, two blog posts. But I also wrote the code for lesson 2, but never finished the blog post.

April–June 2022 (though I'd still like to complete this)

## Non-technical

### Knitted Eeyore toy

![a knitted stuffed Eeyore](eeyore.jpg)

I followed [a pattern](https://www.ravelry.com/patterns/library/eeyore-20) by Claire Garland to knit this stuffed Eeyore toy for my mom (who loves Winnie the Pooh and Eeyore especially) for her birthday. It's the second complete knitting project I've done (the first being a beanie), and the first that uses any stitches other than knit and purl. The pattern is excellent and the knitting was a lot of fun, though I found that I don't enjoy the assembly aspects of knitting a stuffed animal like this; stitching things together, adding the mane, stuffing, etc. all feel a little tedious. That may be because they're more difficult. I was pretty disappointed with how some of my stitches turned out, especially mattress stitch along edges that weren't totally straight. That being said, I'm pretty happy with the overall project and my mom loved it.

January–May 2025
