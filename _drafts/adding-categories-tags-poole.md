---

layout: post
title: "Adding Categories and Tags to the Jekyll Poole Theme"
categories: [tutorial]
tags: [Jekyll, Poole Theme, Markdown, Liquid]
excerpt: Integrating Ctegories and Tags into the Poole Theme for Jekyll

---

### Introduction
Categories and Tags are a common blogging tool.  Jekyll supports them natively, here's how to add them into the Poole Theme.

## Goals
In this tutorial, we will:

* Add tag and category information to the front matter of posts and pages
* Display the categories and tags assigned to a post or page
* Create a page that aggregates tags and categories.
* Create a simple navigation bar (Optional)

## Prerequisites
You'll need to have a couple simple things working already.

### A Web Site Using the Poole Theme for Jekyll
You can use these techniques on another theme, should work, but this tutorial refers specifically to the Poole Theme.

### Text Editor
I use Visual Studio Code, any editor you're comfortable with will work.

### Source Control or an Upload Tool
You can preview the site locally using the `jekyll serve` command but you'll also need to upload the files to your web host.

## Step 1- Adding Categories and Tags to Posts and Pages

Let's start with the easy part.  If you want to use categories and/or tags you'll have to add them to your posts and pages.  Simple enough, open a post up with a text editor and into the front matter (the stuff between the `---` that probably includes `layout:` and `title:`) tags, put something like this

`categories: [category]`  
`tags: [tag, tag, tag, tag]`

> I use a single category to indicate the type of post (Tutorial, Tool, etc.) and one or more tags to indicate what topics it covers (Jekyll, Nagios, Auditing, etc.).  Do what works for you.

Save it up, next we will make each page or post display the categories and tags associated to it.

## Displaying Categories and Tags on Posts and Pages

Open up the `\_layouts\post.html`.

You can put this wherever you'd like, but to put the Categories and Tags assigned to the post underneath the title add the following code in between the `<time datetime="{{ page.date | date_to_xmlschema }}" class="post-date">{{ page.date | date_to_string }}</time>` and `{{ content }}` lines:

``` html 
<div class="post-categories">
    {% if post %}
      {% assign categories = post.categories %}
    {% else %}
      {% assign categories = page.categories %}
    {% endif %}
    Categories:
    {% for category in categories %}
    <a href="{{site.baseurl}}/categories/#{{category|slugify}}">{{category}}</a>
    {% unless forloop.last %}&nbsp;{% endunless %}
    {% endfor %}
  </div>
  ```

To display the tags associated with the post throw the same code in but change `categories` to `tags` and `category` to `tag`.

> I don't use categories or tags in the few pages that I have on the site, but if you want to you can add the same code to the `\_layouts\page.html` file.

If you want to preview the function go to the shell where you've installed Jekyll (i'm using WSL) and type `jekyll serve` then open a browser and go to `http://127.0.0.1:4000` and see how it works.

You'll notice, either in the code or on the preview pages, that there's a link to a "Categories" or "Tags" page.  Let's create those.

## Displaying All Categories or Tags Assigned to Posts or Pages

In the site root directory, create a page called `categories.md`.  Put this code in the page:

``` html
---
layout: page
permalink: /categories/
title: Categories
---

<div id="archives">
{% for category in site.categories %}
  <div class="archive-group">
    {% capture category_name %}{{ category | first }}{% endcapture %}
    <div id="#{{ category_name | slugize }}"></div>
    <p></p>
    
    <h3 class="category-head">{{ category_name }}</h3>
    <a name="{{ category_name | slugize }}"></a>
    {% for post in site.categories[category_name] %}
    <article class="archive-item">
      <h4><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a></h4>
    </article>
    {% endfor %}
  </div>
{% endfor %}
</div>
```

Save it up and preview it at `http://127.0.0.1:4000/categories/`.  You should see a list of all of the categories you've assigned and the pagess assigned those categories.

Do the same thing for tags.  It's all the same, just change `categories` to `tags` and `category` to `tag`.

Out of the box, there isn't a navigation bar that would get you to these two pages easily.  If you want to, you can correct that in the next step.

## Displaying a Simple Navigation Bar
In the `\_config.yml ` file, add:

``` html
pages_list:  
  Categories: '/categories'  
  Tags: '/tags'`
```
> If you want to have other pages appear in the navigation bar you can add them in using the same format.  As an example `About: '/about'` would navigate to the `about.md` or `about.html` file.

That builds the list, now put the navigation bar on every page.  Open the `\_posts\default.html` file.  Add the following code:

``` html
<br>
    {% for page in site.pages_list %}
        &nbsp;&nbsp;&nbsp;
        <small><a href="{{ page[1]  }}">{{ page[0] }}</a></small>
    {% endfor %}`
```
You can place that wherver you like, but I threw it in after the `<small>{{ site.tagline }}</small>` line towards the top of the page.  Save that, preview it.  We're done here.

## Conclusion
Jekyll makes working with categories and tags pretty easy, but you do have to do the work of adding it to the Poole theme.

Play around with the HTML and CSS of the site to get the look you want.

### Resources

* Jekyll Documentation: Posts- https://jekyllrb.com/docs/posts/
* Jekyll Documentation: Navigation- https://jekyllrb.com/tutorials/navigation/
* 3 Simple steps to setup Jekyll Categories and Tags- https://blog.webjeda.com/jekyll-categories/#categories-count

### Revision History
* 02/23/2019- Writing begins