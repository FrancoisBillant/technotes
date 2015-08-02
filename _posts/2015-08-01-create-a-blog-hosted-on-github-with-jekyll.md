---
layout: post
title: "Create a blog hosted on Github with Jekyll"
description: ""
category: [HowTo]
tags: [howto, blog, github, jekyll]
---
{% include JB/setup %}

##Overview

The idea here is to create a blog entirely hosted on github.

To do this I will use:

- Github pages to version and host the blog: [https://pages.github.com/](https://pages.github.com/)
    
- Jekyll and Jekyll-Bootstrap to create and build the blog: [http://jekyllbootstrap.com/](http://jekyllbootstrap.com/)
    
##Requirements

- a github account
- a place to install Jekyll, be it your laptop, a VM or container. You will need to have internet access to be able to push to github   

##Install
I made available on the docker hub a container including jekyll, git and vim, you can find it here: [https://registry.hub.docker.com/u/francois/jekyll/](https://registry.hub.docker.com/u/francois/jekyll/)   
Just run it with:   

    docker run -dti --name jekyll francois/jekyll:latest /bin/bash   
    docker attach jekyll

and you're good to go :)

Otherwise, look at the Dockerfile for the install process ;) [https://github.com/francois-docker/jekyll/blob/master/Dockerfile](https://github.com/francois-docker/jekyll/blob/master/Dockerfile)

##Configuration

###Github

Create a new repository with whatever name you'd like, ie "myblog"   
Go to your workspace (container, vm, laptop...)   

    #Clone the skeleton of the blog   
    git clone https://github.com/plusjade/jekyll-bootstrap.git myblog   
    cd myblog   
    #Change the origin   
    git remote set-url origin git@github.com:USERNAME/myblog.git   
    #Create an orphan branch   
    git checkout --orphan gh-pages

###Blog

- Cleanup

    First, remove examples posts from skeleton   

        rm -Rf _posts/core-samples/

- Custom domain

    TODO

- Config

    Edit the _config.yml file:

        title : Technical Notes
        tagline: To keep track about things IT related...
        author :
          name : somename
          email : somename@somemail.tld
          github : someusername
          twitter : someusername
          feedburner : feedname        

    Comment and fill as suited for your configuration.    
    For example, to use disqus as comment solution:

        comments :
          provider : disqus
          disqus :
            short_name : technotesbillantbzh


- Homepage

    Let's make the homepage more blog like by displaying list of posts.   
    Replace the default index.md by index.html containing the following:

    {% raw %}
        ---
        layout: default
        ---
        
        
        {% for post in site.posts limit:20 %}
            <div class="blog-index">
              {% assign post = site.posts.first %}
              {% assign content = post.content %}
              {% include post_detail.html %}
            </div>
        {% endfor %}
    {% endraw %}

    Then create the _include/post_detail.html file:

        {% raw %}
        <h2 class="entry-title">
        {% if page.title %}
            <a href="{{ root_url }}{{ page.url }}">{{ page.title }}</a>
        {% endif %}
        {% if post.title %}
            <a href="{{ root_url }}{{ post.url }}">{{ post.title }}</a>
        {% endif %}
        </h2>
        <div class="entry-content">{{ content }}</div>
    {% endraw %}

- Create a post

    TODO

- Publish

    TODO

##Usage

- Create a post:   
    TODO
- Create a Draft:    
    TODO
- Publish a draft without plugin T_T:   
    TODO
- Change Theme:    
    TODO


sources:    
[http://jekyllbootstrap.com/usage/jekyll-quick-start.html](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)    
[https://gist.github.com/nimbupani/1421828#file-post_detail-html](https://gist.github.com/nimbupani/1421828#file-post_detail-html)   
[https://truongtx.me/2012/12/27/jekyll-create-a-list-of-lastest-posts/](https://truongtx.me/2012/12/27/jekyll-create-a-list-of-lastest-posts/)   
