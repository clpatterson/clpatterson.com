---
title: Counting Files and Comparing Directories in Linux
description: >-
  I’ve been exploring the /usr directory as a part of a mission to better
  understand the Linux Filesystem Hierarchy Standard — the…
date: '2019-07-12T16:12:42.824Z'
categories: []
keywords: []
slug: >-
  /@Pattersoncharlesl/counting-files-and-comparing-directories-in-linux-bfd3af0a2e8c
---

![](img/1__GINcfpYew8M2__XS9Jubdhg.gif)

I’ve been exploring the /usr directory as a part of a mission to better understand the Linux Filesystem Hierarchy Standard — the topography of the modern server. One question that arose during my travels through the filesystem was how many files are in /bin vs. /user/bin? That’s when I discovered this helpful pipeline.
```
ls | wc -l
```
Pipelines are composed of two or more commands connected with pipes on the command line. The pipe “|” redirects the output of the preceding command into the standard input of the next command, which acts on the data received. In this case, the list of filenames produced by`ls` is passed to `wc -l` which counts the number of newlines in the list (i.e. every filename.) The `wc` command prints the count of newline, words, and bytes of a given file to standard output in that order. Here the `-l` option limits the count returned to just newlines. Below is the command in action.
```
\[user@host /\]$ ls /usr/bin | wc -l  
654
```
There are 654 files in /usr/bin of my Linux distro before any additional software has been installed. How does this compare to /bin? I could run this same pipeline for /bin and compare the result, but that still wouldn’t tell me if these directories have the same contents. Enter `diff`.
```
\[user@host /\]$ diff /usr/bin /bin
```
`diff` compares files line by line and outputs the difference. If the command is silent, then the two files are identical (though this behavior can be changed by using one of `diff`'s many options). I ran diff to compare /usr/bin and /bin and got nothing. But what if they share subdirectories with the same name but different contents?
```
\[user@host /\]$ diff -r /usr/bin /bin
```
Silence. The `-r` option compared my directories recursively and turned up no differences. Now I feel confident the contents are the same, but my original mission to understand /usr is not complete. Onwards!

(Tips, corrections, or comments are always welcome.)