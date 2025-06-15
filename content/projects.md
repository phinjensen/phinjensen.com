---
title: "Personal Projects"
---

I've always got something I'm working on, whether it's every day or once a month. Here are some of the more substantial projects I've worked on.

## Technical

### [rlox — an implementation of Robert Nystrom's Lox in Rust](https://github.com/phinjensen/rlox)

I've been writing an implementation or Lox, following Robert Nystrom's excellent book [Crafting Interpreters](https://craftinginterpreters.com/) through CodeCrafters.io. I haven't done anything interesting to modify the language, but the implementation is different in a lot of ways from the reference Java implementation. I'm currently working through the [Resolving and Binding](https://craftinginterpreters.com/resolving-and-binding.html) chapter.

February 2025–present

### [Koja — private location sharing for Android](https://sr.ht/~phinjensen/koja/)

Koja (کچا, Persian for "where") is an app for privately tracking your location and sharing it with friends and family. I used Google Maps' location sharing for a while, but in the interest of privacy decided to build an app to replicate that functionality. The server is a Supabase project that uses PostGIS for the geographic data, and the client is an Android app written in Kotlin. I got this to a decent point where it fairly reliably sends the location to the server, even auto-starting itself on boot and if it dies (with an easy toggle off, of course). It's almost useful; I just need to push it through some polishing, and then I can actually use it to share my location with my wife.

October 2024–present

### VajehSabt — background listening Persian translation

VajehSabt is an Android app with embedded speech-to-text and speech detection models (both running with PyTorch) that listens for speech and then attempts to transcribe it to Persian text, which it saves to a list that can be viewed later.

For the LING 361: Speech Processing class at BYU, the final project was to build _something_ that used speech processing technology in some way. At the time I was listening to BBC Persian and some other Persian podcasts on my walks to school, and when I came across words I didn't recognize, I would try to write the word down in a notebook and look it up later to figure out what it meant. I chose to make VajehSabt to try to solve that problem for my final project, with the idea that I could start VajehSabt, then a podcast, and just repeat any words I didn't recognize and have it transcribe them for me to look up and memorize later.

I wrote it in Kotlin for Android using Jetpack Compose. I'd done one other Android app for a class, but that was using Java and the older Views UI model, so the language and framework were new. I also had to learn how to integrate PyTorch models into an Android app, for which there are a few tutorials that don't explain certain things enough to make integrating new models very easy (I spent _hours_ trying to figure this out for each model); nevertheless, I did get it working with two different models processing the input from the microphone API.

Unfortunately, the whole thing didn't work out too well in the end. The speech recognition worked, and the speech-to-text _functioned_ but not in a very useful way. Speech-to-text models (at least those I've seen, including Whisper) work best when transcribing whole sentences, not single words, so it incorrectly transcribed the words I said at least 50% of the time.

I think a better idea would be to build an app that can intercept the audio playing through the system and "clip" the audio when the user requests (maybe even using speech detection, e.g. if the user just says "clip" out loud) and saves the audio clip for later and additionally can attempt to transcribe it. That would be more broadly useful and probably work better.

I'd like to open-source this, but I added a lot of models in testing of which I can't remember the license, so I need to clean things up a lot before it's fit for publication.

March–April 2024

### [SeeYouLater — CLI and browser bookmarking tool](https://github.com/phinjensen/seeyoulater)

SeeYouLater is a CLI bookmarking tool with rudimentary sync support and browser extension integration. I initially started it because I tried a tool called [Buku](https://github.com/jarun/buku) but didn't like some of its features (maybe more because I just wanted to start a new project). The idea was to have a fast tool for saving web links that would automatically figure out title and description, have a nice CLI interface, and be easy to sync across computers, but I stopped working on it when I realized I just didn't care that much about managing my bookmarks. I lightly use [https://raindrop.io/](https://raindrop.io/) now and it's Good Enough. And I really don't need (or maybe even want) a CLI-based tool for something that I'll mainly be using with a browser.

It was a fun project while it lasted. It's the first Rust project I used with SQL, and the second project I did with SQLite, and like many other people, I really respect SQLite. It's a great project.

July 2022–November 2023

### [BYU open room finder — tool for finding unused classrooms on BYU campus](https://github.com/phinjensen/byu-tools)

![screenshot showing BYU open room finder](byu-tools.jpg)

BYU has a page[^1] that allows you to view the schedule for each classroom in each building on campus, which is theoretically useful but in practice very difficult to work with if what you're looking for is, for example, _any empty classroom on a Tuesday at 10:30_. To do so, you would have to select a building, click to open a window to get a list of rooms, and then go through that list one by one until you find a room that fits your needs.

This tool makes finding a room much easier. Select a building (or all buildings), day(s) of the week, and a time range, and it will find all of the rooms in that building or on campus that aren't scheduled during that time.

It's a Python+Flask app using PostgreSQL for the database and [Peewee](https://docs.peewee-orm.com/en/latest/) for database access. The scraper uses BeautifulSoup4 and psycopg2.

This used to be hosted at https://byu-tools.fly.dev/ though I haven't kept it running since I graduated.

February 2022-April 2023

### [raytracer — software raytracer written in Rust](https://github.com/phinjensen/raytracer)

![colored balls and triangles with a completely reflective ball at the bottom and harsh light coming from the right](raytracer.jpg)

One project for CS 455: Computer Graphics at BYU was to build a raytracer from scratch. They recommended C++ because the renders could get pretty slow and that's what the TAs were familiar with, but I thought it'd be a good chance to use Rust, and I had a ton of fun with it. We had to output images in the PPM format, support spheres and triangles with diffuse and reflective textures, and allow each ray to bounce at least three times. An interesting mistake I made at one point was incorrectly writing the image size in the output. I didn't notice the mistake because the GIMP happily ignored that metadata and rendered the image without any problem, but the grader used an image viewer that wasn't so loose with the format. I was able to resubmit without any points lost 😁.

This would be a fun project to continue. Things I could add:

- transparent objects
- parallel processing

February–March 2023

### [Ultra Geo Master — a web-based geography game skeleton](https://github.com/phinjensen/ultra-geo-master)

Growing up, my family would play the board game _[Where in the World?](https://boardgamegeek.com/boardgame/9646/where-in-the-world)_—specifically a version where we would take turns drawing a card with various information about a country, reading it to another person, and having them guess where it is on the map. Each country was worth points based on its area; for example, Ukraine is worth 1 point (being the largest country in Europe, excluding Russia, which was in the Asia map/deck), while Vatican City is worth 45.

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

My main creative hobbies are all computer-related, but I'm trying to get more into physical crafts.

### Knitted Eeyore toy

![a knitted stuffed Eeyore](eeyore.jpg)

I followed [a pattern](https://www.ravelry.com/patterns/library/eeyore-20) by Claire Garland to knit this stuffed Eeyore toy for my mom (who loves Winnie the Pooh and Eeyore especially) for her birthday. It's the second complete knitting project I've done (the first being a beanie), and the first that uses any stitches other than knit and purl. The pattern is excellent and the knitting was a lot of fun, though I found that I don't enjoy the assembly aspects of knitting a stuffed animal like this; stitching things together, adding the mane, stuffing, etc. all feel a little tedious. That may be because they're more difficult. I was pretty disappointed with how some of my stitches turned out, especially mattress stitch along edges that weren't totally straight. That being said, I'm pretty happy with the overall project, and my mom loved it. Mission accomplished.

January–May 2025

[^1]: https://y.byu.edu/class_schedule/cgi/classRoom2.cgi
