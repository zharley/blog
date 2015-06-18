---
layout:     post
title:      "Creating this site with Jekyll"
categories: 
---
Creating a site with [Jekyll](http://jekyllrb.com/) is easy, once you've done the hard work of [selecting it from the bevy of other options]({% post_url 2015-04-09-this-is-not-a-blog %}). The main prerequisites are [Ruby](http://www.ruby-lang.org/) (a.k.a. the best programming language in the world) and [Rubygems](http://rubygems.org/), its package manager.

### Getting started

The [Jekyll instructions](http://jekyllrb.com/docs/installation/) say:

> at the terminal prompt, simply run the following command to install Jekyll

    gem install jekyll

Unfortunately, with a plain vanilla installation of **OSX Yosemite**, you are likely to see the following error:

> ERROR:  While executing gem ... (Gem::FilePermissionError)  
> You don't have write permissions for the /Library/Ruby/Gems/2.0.0 directory

This makes sense, actually, because the whole `/Library` directory is owned by the *root* superuser, and contains various system-wide software libraries.

A quick fix would be to throw a `sudo` in front of the command, allowing you to [execute the command as a superuser](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man8/sudo.8.html). I think a better solution is to install [rbenv](https://github.com/sstephenson/rbenv) first, as described in my post about [setting up my development environment]({% post_url 2015-04-17-my-development-machine %}). This is a bit more work, but the payoff is that in the future, you will be able to easily install and manage gems as your regular user (obviating the need for **sudo**). There is a good discussion about [gem installation on Stack Overflow](http://stackoverflow.com/questions/2119064/sudo-gem-install-or-gem-install-and-gem-locations), for further reading.

So, go [install rbenv]({% post_url 2015-04-17-my-development-machine %}), then come back and run:

    gem install jekyll

### New site structure

At this point, assuming you have installed the Jekyll gem (one way or another), create a new directory for your site. In this example, we'll put everything under **blog** with all the Jekyll-specific files in a subdirectory **www**.

{% highlight bash %}
mkdir -p blog/www && cd blog/www
{% endhighlight %}

Within this directory, run:

{% highlight bash %}
jekyll new .
{% endhighlight %}

You should see a message like *New jekyll site installed,* and find various files and directories that Jekyll just generated:

    .
    ├── _config.yml
    ├── _includes
    │   ├── footer.html
    │   ├── head.html
    │   └── header.html
    ├── _layouts
    │   ├── default.html
    │   ├── page.html
    │   └── post.html
    ├── _posts
    │   └── 2015-04-23-welcome-to-jekyll.markdown
    ├── _sass
    │   ├── _base.scss
    │   ├── _layout.scss
    │   └── _syntax-highlighting.scss
    ├── about.md
    ├── css
    │   └── main.scss
    ├── feed.xml
    └── index.html

The most important directory is **_posts**, which stores your content (currently consisting of one auto-generated post). For more details, consult [the documentation covering the directory structure](http://jekyllrb.com/docs/structure/).

### Running Jekyll locally

At this point, you already have a working site! To see it, run:

{% highlight bash %}
jekyll serve
{% endhighlight %}

Then use your browser to point to [http://127.0.0.1:4000/](http://127.0.0.1:4000/), and you should see something like:

[![Initial Jekyll site](/assets/images/2015-04-23-174635-initial-jekyll-site.png "Initial Jekyll site")](/assets/images/2015-04-23-174635-initial-jekyll-site.png)

The initial auto-generated post looks like:

[![Auto-generated post](/assets/images/2015-04-23-174749-auto-generated-post.png "Auto-generated post")](/assets/images/2015-04-23-174749-auto-generated-post.png)

### Posting something new

Creating a new post is as easy as creating a new file in the **_posts** subdirectory. Start by copying the example post:

{% highlight bash %}
cp _posts/*welcome-to-jekyll.markdown _posts/`date +%F`-my-new-post.markdown
{% endhighlight %}

This uses the `date` shell command to create a `YYYY-MM-DD` formatted datestamp. In my case, this created a new file called `2015-04-23-my-new-post.markdown`. Edit this file, especially the top part, so that the title is different:

{% highlight yaml %}
---
layout: post
title:  "My new post!"
date:   2015-04-23 17:45:06
categories: jekyll update
---
{% endhighlight %}

After saving, go back to [http://127.0.0.1:4000/](http://127.0.0.1:4000/), and you should see both posts listed:

[![New post appears](/assets/images/2015-04-23-180003-new-post-appears.png "New post appears")](/assets/images/2015-04-23-180003-new-post-appears.png)

For more details on editing posts, consult the [Jekyll documentation about posting content](http://jekyllrb.com/docs/posts/). It's also nice to have a reference to [Markdown syntax](http://daringfireball.net/projects/markdown/syntax) handy.

### Next steps

You can deploy a Jekyll site practically anywhere, because it's static. To build your site for deployment, run:

{% highlight bash %}
jekyll build
{% endhighlight %}

All of the distributable (deployment) files end up in the **_site** subdirectory, which you can upload to a host of your choice. The documentation lists some reasonable [methods for deployment](http://jekyllrb.com/docs/deployment-methods/). Another cool method is [hosting via GitHub Pages](http://jekyllrb.com/docs/github-pages/) which is especially nice if you're already on GitHub.

I happen to be using a [static application](https://docs.webfaction.com/software/static.html) on [Webfaction](https://www.webfaction.com/).
