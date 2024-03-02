---
title: Making the State Department Travel Advisory Map Better
author: Phineas Jensen
date: 2024-03-01
description: "Annoyance—Idea—Implementation"
tags:
  - typescript
---

In my recurring fits of wanderlust, I enjoy looking through the United States State Department [Travel Advisories](https://travel.state.gov/content/travel/en/traveladvisories/traveladvisories.html/). It's fun to see what countries are considered safe for travel by the US government and which aren't. They categorize countries into four levels of advisory: "Exercise Normal Precautions", "Exercise Increased Caution", "Reconsider Travel", and "Do Not Travel". These are pretty self-explanatory and many aren't surprising. For example, Canada falls under level 1, (*Exercise Normal Precautions*), while Ukraine, being a war zone right now, is level 4 (*Do Not Travel*).

Some classifications are more surprising to me:

- Much of Western Europe (including Germany, France, and Italy) gets a level 2 advisory, while much of Eastern Europe (Bulgaria, Romania, Poland) only gets level 1. I've traveled some in both sides of Europe and would have assumed they would have the same advisory levels, or perhaps that Eastern Europe would have a slightly higher advisory level, perhaps due to health concerns in slightly poorer countries. In fact, that's not the case, with *terrorism* being the reason for the increased caution in Western Europe.
- Botswana and Zambia are both level 1. I readily admit that Africa is a gap in my knowledge and my perception of it has been influenced by western media, leading me to assume that *all* of Africa must be dangerous to some extent. Obviously, that's wrong, and these advisories helped me to start to see that.
- None of the four current [Communist countries](https://en.wikipedia.org/wiki/List_of_communist_states) of the world receive a level 4 (*Do Not Travel*) advisory. Vietnam is level 1, Cuba and Laos are level 2, and China is level 3. I had always assumed that most or all of these would have higher advisory due to the historically fraught relationship the US has had with Communism.

Anyway, I like looking through these advisories, and I love maps, so I was happy to find that the state department [provides](https://travelmaps.state.gov/TSGMap/) a color-coded map showing all of these advisories. Unfortunately, it's quite slow, taking ~20 seconds to load on my laptop and transferring 8 MB of data, it's quite ugly (in my humble opinion), and the interface doesn't readily show why a country gets the classification it has (navigation to another page is required).

I've been wanting to try using [maplibre-gl-js](https://github.com/maplibre/maplibre-gl-js)[^1] with some vector tilesets and data, so I decided it was a perfect opportunity to learn that library and make a map that is better. The US Gov provides the advisory data as an [XML file](https://catalog.data.gov/dataset/travel-alerts-3417f), and the wonderful public-domain [Natural Earth](https://www.naturalearthdata.com/) project provides high-quality data for country shape data. I wrote a build script to map the two together and generate a GeoJSON file which stores the relevant data in properties, and then created a basic map page which uses that data to display the countries with colors relating to their advisory levels, and a nice sidebar that displays all of the advisory info when a country is clicked.

Here's a small demo video:

<video width="100%" autoplay controls muted>
  <source src="/blog/2024/travel-map-demo.webm" type="video/webm">
  Your browser does not support the video tag.
</video>

And, of course, please [try the map out for yourself](https://maps.phinjensen.com/)! I think it was a success. I find it much nicer to use than the official map. It's roughly 4x faster to load (only ~5 seconds for a complete render on my laptop vs 20 s for the official map) and lighter (only 2.36 MB transferred instead of 8), and more importantly, it *feels* much more responsive (MapLibre and vector tiles are awesome technologies!). I also find the sidebar with all of the advisory information much nicer than having to click on a link, although there is a link to the State Department's page for each country if desired.

Like most of my projects, this one is [open source](https://github.com/phinjensen/maps).

[^1]: A great library with a name that isn't so fun to type.
