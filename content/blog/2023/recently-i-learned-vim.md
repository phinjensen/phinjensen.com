---
title: "Recently I Learned: (Neo)Vim"
author: Phineas Jensen
date: 2023-06-03
description: "Moving a line to the top, middle, or bottom of view—A shortcut for match in s///—Replacing newlines with literal carriage returns—Quickly diffing files"
tags:
  - vim
  - neovim
---

In the great buffet of built-in Vim and Neovim features, I’ve recently added to my plate a few that I’ve really been enjoying or found indispensable for my specific use cases. In the spirit of Luke 6:31[^1], I thought I’d share them with the world.

## Moving a line to the top, middle, or bottom of the view

I often find when in the midst of software archaeology—or trying to simply see all of what I just wrote—that I’d like to be able to quickly shift the editor’s view to so that a given line is at the top. For example, I’m casually reading code and reach a new function header:

```rust
    // lots
    // of
    // lines
    // of
    // code
    // which
    // i
    // no
    // longer
    // care
    // to
    // look
    // at
|   fn super_interesting_function() { // See my cursor on this line?
        // code I'd
        // like to see
```

I’d like to be able to make `super_interesting_function` be at the top of the screen so I can see its entire body at once without pressing `j` or `Ctrl+E` dozens of times or counting the number of lines I want to scroll down and pressing `13j`. Having identified this problem, I did a search and learned about a few commands that I’ve used very frequently since then: `zt`, `zz`, and `zb`. Respectively, they will put the line on which your cursor is at the top, center, or bottom of the screen. `zt` especially has been invaluable in the few short weeks I’ve been using it.

These commands each have a counterpart that also moves the cursor to the first non-blank character of the line: `z<CR>` (that is, `z` followed by the Enter key), `z.`, and `z-`. I don’t find myself using these as much, but they’re good to keep in mind.

All of these can be preceded by a number to specify a line to apply the command to. I find that useful when my cursor is near the top of the screen, I see a function definition on (for example) line 216, and I’d like to move that line to the top of the screen. I can just type `216zt` and my wish is granted.

These commands are documented at [`:help scroll-cursor`](https://vimhelp.org/scroll.txt.html#scroll-cursor).

## Using the matched text in an `s///` command

Vim’s `sed`-like search-and-replace feature is one that I use constantly. I use it constantly to reformat data to match the syntax of a programming language, to rename functions, to remove whitespace at the end of a line, and many other tasks. I often have found myself wanting to be able to essentially keep what I’m matching and add some text around it, though. I didn’t bother to look up, possibly for years, whether there was a way to do that, and recently and blessedly stumbled my way into finding it while looking for information on the following tip (it’s just not as exciting) and found it at [`:help s/\&`](https://vimhelp.org/change.txt.html#s%2F%5C%26) (I didn’t go directly to that page, but it’s the most direct link).

Basically, you can use the character `&` to in the replacement portion of an `s///` substitution to stand in for the entire match. So I could quickly convert a newline-separated list of words (for example, something copied from a browser) into a JavaScript array of strings by `VISUAL LINE` selecting the words and using the following substitution:

```
s/\w\+/    "&",/
```

...with indentation to boot.

Another small thing, but one that makes me really happy.

## How to add literal carriage returns

This probably won’t apply to many people at all, but for work I’ve recently been working on some healthcare-related software, and most healthcare systems (in the US, at least) seem to use a message format called Health Level Seven, or HL7 for short. This is a somewhat odd but fairly easy to parse format. It consists of a series of segments separated (sort of) by lines. The problem is that the lines are delimited by carriage returns (`\r`, ASCII 0x0d) rather than the newlines (`\n`, ASCII 0x0a) that Unix systems use or even the combination `\r\n` that Windows and HTTP use.

Thus, I’ve found it hard to get software to use or even produce these characters correctly. When saving an HL7 message which incorrectly has `\n` as the line delimiter to a file, I tried using `sed` to convert them to carriage returns, but no luck. Perhaps there’s a flag I couldn’t find or, I just did something wrong, but I couldn’t get it to work. Likewise, using `s/\n/\r/g` in Vim didn’t do the trick. That’s when I stumbled upon the list of special characters in a replacement string; specifically [`\<CR>`](https://vimhelp.org/change.txt.html#s%2F%5C%3CCR%3E). Rather than literally typing `<CR>`, that character is inserted by typing `Ctrl+V <Enter>`. It’s a little odd, but it makes a lot of sense intuitively—`<CR>` (through the `Ctrl+V <Enter>` trick) is how you represent a normal newline, which allows splitting lines in a search-and-replace, and adding the `\` makes it a literal `\r` instead. It’s definitely a little confusing that the character called `<CR>` in the documentation isn’t actually a carriage return character, but oh well. Problem solved.

## Viewing a diff for files I already have open

Vim has a nice feature called `vimdiff`, which does a diff of two files (if you’re not familiar with diff, it highlights the differences between two files) within the editor, showing them side-by-side, keeping lines in sync, and making it easy to navigate because all of vim’s normal keybindings are available. The [documentation](https://vimhelp.org/diff.txt.html#start-vimdiff) points out that this can be started easily by using the `vimdiff` command or simply invoking `vim` with the `-d` flag, but sometimes I’ve already got the files open that I want to compare.

As it turns out, there’s an easy way to compare them without re-opening the editor. Just run `:difft` (or the verbose `:diffthis`) in each buffer you’d like to compare and you’ll get a nice color-highlighted comparison of the two files. In my Neovim setup, this looks fantastic, and it even folds large stretches of lines which are the same, making it really easy to see only what changed. In the default, unconfigured version of vim on Fedora that I just installed to test it, it’s hideous, but I’m sure that can easily be fixed.

## Conclusion

There isn’t much of one. I just liked these, and dear reader, I hope you do too. Note that while I mostly refer to vim, these all work in Neovim as well. Go forth and edit text.

[^1]: “Do to others as you would have them do to you.” (New English Translation)
