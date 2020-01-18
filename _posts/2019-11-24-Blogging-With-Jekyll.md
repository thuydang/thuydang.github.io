---
layout: post
comments: true
title: You're up and running!
feature-img: "assets/img/pexels/book-glass.jpeg"    # Add a feature-image to the post
#thumbnail: "assets/img/thumbnail/desk-messy.jpeg"   # Add a thumbnail image on blog view
categories: [blog]
tags: [jekyll]
series: "Blogging with Jekyll"
---

## About this series 
{% include series.html %}
----

Next you can update your site name, avatar and other options using the _config.yml file in the root of your repository (shown below).

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. This article provides a quick guide on using Jekyll for writing a blog article. For more instructions about setting up this blog head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.

* [Quick markdown guide](https://raw.githubusercontent.com/barryclark/www.jekyllnow.com/gh-pages/_posts/2014-6-19-Markdown-Style-Guide.md).
* [Jekyll docs](https://jekyllrb.com/docs/posts/).

## How to write an article

### Draft
Very easy. Create a post in _drafts folder!.

## Troubleshooting

### Code snippet latex

All the normal tricks with raw tag do not seem to work with latex code snippet. There is this error:

```
{% raw %}
Liquid Exception: Liquid syntax error (line 31): Tag '{%' was not properly terminated with regexp: /\%\}/ in /data/workspace/thuydang.github.io/_posts/2020-01-18-Latex-Tricks-for-Paper-Publication.md
{% endraw %}
```
The indicated line points to line 7 of the following snippet, which seems fine.

{% highlight latex linenos %}
\begin{figure*}[th!]
  \centering
  \begin{minipage}[t]{\linewidth}
    \includegraphics[width=0.90\textwidth,height=5.9cm]{images/arche_overview}
  \caption{Multi-domain network slicing management architectures}
\label{fig:multi_domain_arche}
%\parbox{6.5cm}{\small \hspace{1.5cm} }
\end{minipage}
\end{figure*}
{% endhighlight %}

To debug, a local ruby jekyll environment is installed. 

1. Install Jekyll and plug-ins in one fell swoop. gem install github-pages This mirrors the plug-ins used by GitHub Pages on your local machine including Jekyll, Sass, etc. `ruby-devel` and etc. may be required.
2. Clone down your fork `git clone https://github.com/yourusername/yourusername.github.io.git`.   
3. Serve the site and watch for markup/sass changes `jekyll serve`.
4. View your website at <http://127.0.0.1:4000/>.
5. Commit any changes and push everything to the master branch of your GitHub user repository. GitHub Pages will then rebuild and serve your website.


**Solution:**

1. Remove all indication of language highlight, just keep the fence block. Compile pages. Add programming language to the fence block. It works magically!!!

OR 

2. Wrap the code around hightlight - endhighlight tags. Now language and linenos (line number) can be set.

~~~
{% raw %}
{% highlight latex linenos %}

{% endhighlight %}
{% endraw %}
~~~

## Building the blog

### Search tools

* We use this: <https://digitaldrummerj.me/blogging-on-github-part-7-adding-a-custom-google-search/>
* Others: <https://github.com/jekylltools/jekyll-tipue-search>
