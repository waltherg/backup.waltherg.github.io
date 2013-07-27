---
layout: post
title: "Extensible Fetch of DOM Elements Depending on URL"
date: 2013-07-26
categories: javascript programming
---

For a [project](https://github.com/waltherg/articlEnhancer) I sometimes
tinker with, I need to extract specific DOM elements on web pages but only
if the URL is one in a certain list.

More specifically, I am interested in the *title* element of the abstract or
fulltext view of articles posted on the websites of journals
and repositories.
By the latter I mean places such as PubMed and arXiv.

In practice, I would go to a website and view the source code of the HTML
file for an arbitrary article to figure out which HTML element is *visually*
the title of that page.

As an example, the source code of [this article](http://www.ncbi.nlm.nih.gov/pubmed/23880940) on PubMed shows that the title is located in an `<h1>` element:

{% highlight html %}
<h1>Par6 is phosphorylated by aPKC to facilitate EMT.</h1>
{% endhighlight %}

Further inspection of the same page tells us that there are two `<h1>` elements
and looking at a small sample of articles on PubMed convinces us that the title
is always found in the second `<h1>` element (this is different for PubMed's
new [PubReader](http://www.ncbi.nlm.nih.gov/pmc/about/pubreader/) format).
Translating this to JavaScript code, we can get a pointer to the title element
of articles posted to PubMed as follows:

{% highlight js %}
if (hostname.indexOf("ncbi.nlm.nih.gov") != -1){
     var h1 = document.getElementsByTagName('h1');
     title = h1[1];
}     
{% endhighlight %}

The `indexOf` method invoked on `hostname` (`hostname` is an String object)
checks if there is any overlap between the string stored in `hostname` and
`ncbi.nlm.nih.gov`. If there is no overlap then we know we are not looking
at a PubMed page and `indexOf` returns the value -1.

This procedure varies of course between different journals, and in fact, even
varies for different views of PubMed articles (see the above reference to
PubReader).

My goal here is simple:
Modularize this procedure so that the hostname check and the commands
necessary to retrieve a pointer to the title element can be stored in a
configuration file separate from the main code of this project.

Let's encode a more robust variant of the above procedure (to account for
the PubReader view) in a `json` structure and add a procedure for PLoS journals:

{% highlight js %}
json_hosts = {
  "names": [
        "ncbi.nlm.nih.gov", 
        "plos"
   ],
  
   "plos": ["title = document.getElementsByTagName('h1')[0];"],
   "ncbi.nlm.nih.gov": [
        "var h1 = document.getElementsByTagName('h1');",
        "for(var i=0; i<h1.length; i++){",
        "if(h1[i].className == \"content-title\"){",
        "console.log(h1[i]);",
        "title = h1[i];",
        "}",
        "}",
        "if ( title == null )",
        "title = h1[1];"
   ] 
};
{% endhighlight %}

Then we can extract the title element like so:

{% highlight js %}
function get_title_element(hostname){
    var title = null;
    var host = null;
    
    for(i in json_hosts["names"])
        if(hostname.indexOf(json_hosts["names"][i]) != -1)
           host = json_hosts["names"][i];
    
    var command = "";
    for(i in json_hosts[host])
        command += json_hosts[host][i] + "\n";
        
    if (command.length > 0)
        title = eval(command);

	return title;
}
{% endhighlight %}

The above method is very simple:
first retrieve the `host` label we defined in `json_hosts`, then
extract the JavaScript commands line-by-line from the corresponding
element in `json_hosts`.
At last, execute the assembled command with `eval` and return the
retrieved title element.

Yes, my current solution for this problem uses `eval` and
[I should probably be ashamed of myself](http://blogs.msdn.com/b/ericlippert/archive/2003/11/01/53329.aspx).
However, we know exactly what commands will be executed by `eval` --
i.e. those defined in `json_hosts`. So from a security point of view,
using `eval` should be fine here.

If there is an alternative solution that does not use `eval` is yet to be seen.

In the meantime, please feel free to extend the list of journals and hosts
that are supported by [articlEnhancer](http://waltherg.github.io/articlEnhancer/)
by editing [the hosts file](https://github.com/waltherg/articlEnhancer/blob/master/hosts.js)
as shown here.
