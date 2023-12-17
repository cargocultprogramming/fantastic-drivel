---
layout: post
title: Showing directory structure with tree
description: Most important tree options and commands to show directory structures
tags: cli linux
categories: 
featured: false
---

`tree` is a command line utility to display directory structures in ascii-art-style.

It's got a ton of configuration options. Here are the most useful options:

Option `-L`
 : `-L n` allows to specify the number of levels that tree will display. This does _not_ limit the number of files, so be careful if the directory tree contains folders with lots of files.

 Option `--gitignore`
 : Will use a .gitignore file, if present and include the contained rules to filter the result. However, directories and files starting with `.` are not shown by default.

 Option `-d`
  : Will only show directories, not files.
