---
layout: post
title: "Adding Simple Search to your Jekyll Blog"
author: Evan Kuhn
date: 2017-06-22 17:25:00 -0400
categories: webdev
summary: How to add simple search functionality to your Jekyll blog.
---

## Overview

Today I added simple search functionality to this blog, and I wanted to document how I did so.  It's quite easy, thanks to the [Simple-Jekyll-Search](https://github.com/christian-fei/Simple-Jekyll-Search) library by Christian Fei.

The `Simple-Jekyll-Search` library works by constructing a `search.json` file containing data for all pages and posts in your site.  The JavaScript code then parses through this JSON on the client side. While this is fine for a small site, anything larger will require building an index of your site (eg: via Solr or ElasticSearch) and using the index to construct a set of results. That's more complicated, though.

Also, keep in mind that I've kept full-text search disabled, though you could enable it. But then you'd essentially be transferring your entire site's contents with each request! I compared the size of the `search.json` file with and without full-text, and found:

- basic search: 4 KB
- full-text: 60 KB

That's quite a difference, so I left full-text disabled.

## Installation

I found the installation instructions for `Simple-Jekyll-Search` to be a bit complex, so I've simplified them.

First, we grab the necessary files.  From the root directory of your Jekyll blog:

```bash
# Create a js/ dir
mkdir -p js/simple-jekyll-search

# Copy the necessary files
GIT_URL='https://raw.githubusercontent.com/EvanKuhn/evankuhn.github.io/master'
wget -P js/simple-jekyll-search ${GIT_URL}/js/simple-jekyll-search/LICENSE.md
wget -P js/simple-jekyll-search ${GIT_URL}/js/simple-jekyll-search/jekyll-search.js
wget -P js/simple-jekyll-search ${GIT_URL}/js/simple-jekyll-search/jekyll-search.min.js
wget -P js/simple-jekyll-search ${GIT_URL}/js/simple-jekyll-search/search.basic.json
wget -P js/simple-jekyll-search ${GIT_URL}/js/simple-jekyll-search/search.fulltext.json

wget -P _includes ${GIT_URL}/_includes/search-box.html
wget -P _includes ${GIT_URL}/_includes/search-code.html

# Select the type of search you want (basic vs fulltext)
cp js/simple-jekyll-search/search.basic.json search.json
```

Next, include the `search-box.html` and `search-code.html` HTML snippets in the proper place (eg: in your `default.html` layout):

```html
{% raw %}<!DOCTYPE html>
<html lang="en-us">
  {% include head.html %}

  <body>
    {% include header.html %}

    <section class="main-content">
      {% include search-box.html %}     <!-- ADD THIS -->
      {{ content }}
      {% include footer.html %}
    </section>

    {% include search-code.html %}      <!-- ADD THIS -->
  </body>
</html>{% endraw %}
```

Now refresh your site and try searching. You should see the results list auto-complete with each keystroke:

<div class="centered-image">
<img src="/images/sample_search_results.png" alt="">
<p>Search will results auto-populate under the text box.</p>
</div>

Finally, note that I slightly modified the JavaScript code to clear the search results when the text box is empty, or when you press `Esc`.
