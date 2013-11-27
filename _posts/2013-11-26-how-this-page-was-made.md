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

* Render mathematical equations written in $\LaTeX$,
* visual tagging system to mark certain posts as work-in-progress
  while I still work on them, and
* continuous row labels for all text and code rows in each blog post.

## Equations in LaTeX

While MathJax is an obvious choice for this, there exist some parsing
problems when incorporating LaTeX equations in Markdown documents that
get parsed by Jekyll.

This [blog post](http://cwoebker.com/posts/latex-math-magic) describes the
problem and suggests a solution:
in short, by default Jekyll would misinterpret some $\LaTeX$ syntax as
Markdown syntax and attempt to parse it accordingly.

Once this was sorted out, I needed to decide on the following issue:
do I prefer that $\LaTeX$ equations are rendered as quickly as possible
but the overall page load is slower, or
do I prefer that the page load is as quick as possible with $\LaTeX$
equations rendered slowly?

To give priority to $\LaTeX$ rendering I need to load the MathJax
JavaScript file at the top of the page, whereas to give
priority to the overall page (text, images, etc.) I need to load
MathJax at the bottom of the page,
[see this discussion](https://groups.google.com/forum/#!topic/mathjax-users/Zcq0hTp0VHY).

Putting this together, I now have the following block of JavaScript
code at the top of my html pages:

{% highlight html %}
    <script type="text/x-mathjax-config"> 
    MathJax.Hub.Config({ 
        jax: ["input/TeX","output/HTML-CSS"], 
        extensions: ["tex2jax.js"], 
        tex2jax: { 
        inlineMath: [ ['$','$'], ["\\(","\\)"] ], 
        displayMath: [ ['$$','$$'], ["\\[","\\]"] ], 
        processEscapes: true 
        }, 
    }); 
    </script> 
    
    <script type="text/x-mathjax-config">
    MathJax.Hub.Config({
        tex2jax: {
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre']
        }
    });

    MathJax.Hub.Queue(function() {
        var all = MathJax.Hub.getAllJax(), i;
        for(i=0; i < all.length; i += 1) {
            all[i].SourceElement().parentNode.className += ' has-jax';
        }
    });
    </script>

    <script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js"></script>
{% endhighlight %}

The first block of code, 
[from here](https://groups.google.com/forum/#!topic/mathjax-users/Zcq0hTp0VHY),
defines delimiters that tell MathJax what
parts of the Jekyll-generated html file are $\LaTeX$ equations.

The second block of code,
[from here](http://cwoebker.com/posts/latex-math-magic),
tells MathJax to ignore all text enclosed in certain html tags such as
those for inline JavaScript.
The same block of code adds a class named "has-jax" to all elements
in the page that were identified as $\LaTeX$ snippets that MathJax should
process - this allows us to modify how these equations are displayed in our
CSS file.

The last block of code is just the common mode of loading the current
version of MathJax from a content distribution network (CDN), as
[described here](http://docs.mathjax.org/en/latest/configuration.html#loading-cdn).

**Note** that the two blocks of code that configure the behavior of
MathJax apparently need to precede loading MathJax.
I noticed that MathJax appears to ignore these two blocks of code
if they appear in the html page after loading the MathJax JavaScript file.

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
