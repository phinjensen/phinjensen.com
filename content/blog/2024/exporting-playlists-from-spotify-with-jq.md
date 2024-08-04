---
title: >
  TIL: Exporting Spotify Playlists to YT Music with jq and read
author: Phineas Jensen
date: 2024-08-03
tags:
  - til
  - cli
---

Since my wife Alayna and I recently graduated from college, we no longer benefit from student discounts. This has led to us changing or canceling various subscriptions, including her Spotify subscription, which didn't seem to pay for itself since she hasn't bee listening to music more than a few times a week. Instead, she's been using their free tier to shuffle her existing playlists or just look videos up on YouTube, but both of those options are riddled with ads. It's not a huge deal, but it can be annoying to be shuffling hymns on Sunday morning and get interrupted by an ad for insurance or something.

I still maintain a YouTube Premium subscription as part of a family plan with my parents and siblings, so I added my Google account to her phone so she can use YT music. It's been working well, except my account doesn't have her playlists. She's got a dozen or so that she uses regularly, some of which have over 100 songs, and it would be a pain to copy them over manually. Sounds like a great opportunity for some automation!

A quick search showed several online platforms that will export playlists from one platform to another; they tend to be paid products. Instead, I found the [spotify_to_ytmusic](https://github.com/linsomniac/spotify_to_ytmusic) project which has some clear instructions on how to do exactly what I wanted. The only issue I came across is that it _doesn't_ allow you to choose exactly which playlists to export. We only wanted to export some, since some of them were year-end playlists or made for other people. However, this project _does_ have a script that will export data about every playlist in an account to JSON, which can then be manipulated into just what you want. Here's what I did:

1. Used the login scripts for Spotify and YT Music
2. Ran the export script to get a JSON file of all of Alayna's playlists
3. Had Alayna tell me which playlists she wanted exported
4. Used vim macros to remove all playlists that she didn't want from the exported JSON. If you're curious:
   1. I navigated to the first playlist object's opening `{`.
   2. Recorded a macro to the register `q`: `da{dd`, which deletes the object bound by curly brakets, then deletes the line which contains a dangling comma.
   3. Used `%j` to jump from one playlist to another, and called the macro (`@q`, then just `@@`) on each object I wanted to delete.
5. Then I learned a bit of `jq` to make a parseable list of only the name and IDs of those lists:
   ```
   jq -r '.playlists.[] | [.name, .id] | @tsv'
   ```
6. Once I confirmed that worked, I learned how to use `read` to iterate over each line with the name and ID in variables:
   ```
   jq -r '.playlists.[] | [.name, .id] | @tsv' | while IFS=$'\t' read name id
   do
       echo $name $id
   done
   ```
7. Then I used the `s2yt_copy_playlist` script to copy each of those playlists from Spotify to YT Music, slightly changing the name to avoid merging playlists:
   ```
   jq -r '.playlists.[] | [.name, .id] | @tsv' | while IFS=$'\t' read name id
   do
       s2yt_copy_playlist $id "+(Alayna) $name"
   done
   ```

And that worked! I'm pretty pleased with it. It took a bit more trial and error than I wrote about to get the playlist names working correctly, but it wasn't bad at all. I think some songs may not have imported correctly, but I haven't looked too closely. `jq` is a nice tool that I've been meaning to learn, so I was glad to get one step closer to "knowing" it.

P.S. It would have been much easier to use `jq` to export the names and IDs into a file and edit that rather than using vim macros to edit a massive JSON file, but it wasn't too bad either way.
