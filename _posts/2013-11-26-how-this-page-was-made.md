---
layout: default
title: "How This Page Was Made"
date: 2013-11-26
tags: wip
---

**This post is work in progress (WIP)**

# How This Page Was Made

I am in the process of building my personal website that I intend to use
mostly as a blog.
I chose [Jekyll](http://jekyllrb.com/) in combination with
[GitHub Pages](http://pages.github.com/), which is a true and tested solution
for personal and scientific blogs by now.
There are countless fantastic tutorials that cover setting up a blog
with Jekyll and GitHub Pages, for instance
[Cecil Woebker](http://cwoebker.com/posts/jekyll-blogging)'s post.

Before building this website, I knew I would want my blog to have
the following technical features:

* Render mathematical equations written in LaTeX,
* visual tagging system to mark certain posts as work-in-progress
  while I still work on them, and
* continuous row labels for all text and code rows in each blog post.

## Equations in LaTeX

While MathJax is an obvious choice for this, there exist some parsing
problems when incorporating LaTeX equations in Markdown documents that
get parsed by Jekyll.

This [blog post](http://cwoebker.com/posts/latex-math-magic) describes the
problem and suggests a solution.

## Visual Tags

Jekyll supports tags by default as specifications in the
[YAML front matter](http://jekyllrb.com/docs/frontmatter/).
You can read out and collect tags used in your blog posts with
a Ruby plugin as described in this 
[tutorial](http://charliepark.org/tags-in-jekyll/).

What I need for this point on my features list is that I can get Jekyll
to render the tag *WIP* as a visual badge or similar.

## Line Numbering

Naturally, most tutorials on numbering lines cover line numbers in
code segments.
