---
layout:     post
title:      "Presentation tools for programmers: Reveal.js"
categories: html5 css3
---
The last 5 years have seen a proliferation of browser-based presentation tools, fuelled by [widespread HTML5 and CSS3 adoption](http://html5readiness.com/). A great example is [Reveal.js](https://github.com/hakimel/reveal.js), which lets you make impressive animated slide shows with nothing more than native browser capabilities ([see demo](http://lab.hakim.se/reveal-js/#/)).

### The field of presentation tool alternatives

Anecdotal evidence and [Google Trends]("https://www.google.ca/trends/explore#q=%2Fm%2F0k7vl%2C%20%2Fm%2F01qkry%2C%20%2Fm%2F076tblq&cmpt=q&tz=Etc%2FGMT%2B5" "Google Trend for PowerPoint") suggests that [PowerPoint](https://products.office.com/en-ca/powerpoint) is still king, with [Prezi](https://prezi.com/) and [Keynote](http://www.apple.com/ca/mac/keynote/) making limited inroads.

![Google Trend for PowerPoint](/assets/images/2015-12-01-061434-google-trend-for-powerpoint.png "Google Trend for PowerPoint")

Part of [the long tail](http://www.thelongtail.com/about.html) consists of dozens, maybe hundreds of in-browser slide presentation frameworks and services. Many are for the general public, like [Google Slides](https://www.google.ca/slides/about/), which is [easy to use](https://www.youtube.com/watch?v=sN-POV5ddDs). Others are for programmers:

* [Impress.js](https://github.com/impress/impress.js) ([demo](http://impress.github.io/impress.js/#/bored)): A Prezi-inspired toolkit for making non-linear presentations.
* [Cleaver](https://github.com/jdan/cleaver) ([demo](http://jdan.github.io/cleaver/)): Takes pure Markdown and makes no-nonsense, "for hackers" slides.
* [Deck.js](https://github.com/imakewebthings/deck.js) ([demo](http://imakewebthings.com/deck.js/)): A complete, well-documented JavaScript API and a system for extensions (e.g. [a Markdown extension](https://github.com/tmbrggmn/deck.js-markdown)).
* [Shower](https://github.com/shower/shower) ([demo](http://shwr.me/)): Another very nice HTML5 presentation framework, perhaps not as actively developed or well-known as some of the others.
* [Bespoke.js](https://github.com/bespokejs/bespoke) ([demo](http://bespokejs.github.io/bespoke-theme-cube/)): A minimal system (only 1KB worth of JavaScript) with an admirable focus on modularity and flexibility.

In the end, I liked **Reveal.js** because it's fully-featured and has out-of-the-box support for [Markdown](http://daringfireball.net/projects/markdown). Here's an [introduction to Haskell](http://shuklan.com/haskell/lec07.html) that uses **Reveal.js**.

### Getting started with Reveal.js

[Setup is easy](https://github.com/hakimel/reveal.js/#installation), but I'll just recap it here (note that this is the "full" rather than "basic" setup).

First, install [Node.js](http://nodejs.org/). On OSX, that's as easy as:

{% highlight bash %}
brew install node
{% endhighlight %}

Next, install [Grunt](http://gruntjs.com/getting-started#installing-the-cli).

{% highlight bash %}
npm install -g grunt-cli
{% endhighlight %}

Clone the **Reveal.js** repository.

{% highlight bash %}
git clone https://github.com/hakimel/reveal.js.git
{% endhighlight %}

Enter the diretory and install dependencies.

{% highlight bash %}
cd reveal.js
npm install
{% endhighlight %}

Finally, launch the server and navigate to <http://localhost:8000>:

{% highlight bash %}
grunt serve
{% endhighlight %}

You should see this:

![Default Reveal.js presentation](/assets/images/2015-12-03-205534-default-reveal-js-presentation.png "Default Reveal.js presentation")

### Authoring slides

A presentation's contents are captured in a single `index.html` file. The following is a single slide, composed in HTML:

{% highlight html %}
<section>
  <h2>Touch Optimized</h2>
  <p>Presentations look great on touch devices, like mobile phones and tablets. Simply swipe through your slides.</p>
</section>
{% endhighlight %}

Using optional Markdown, a slide looks like this:

{% highlight html %}
<section data-markdown>
  <script type="text/template">
    ## Markdown support
    Write content using inline or external Markdown.
    Instructions and more info available in the [readme](https://github.com/hakimel/reveal.js#markdown).
  </script>
</section>
{% endhighlight %}

While it's running, the **grunt server** command automatically refreshes your browser when it detects changes in the `index.html` file, thanks to [live reload](https://github.com/gruntjs/grunt-contrib-connect). This makes it convenient to edit slides and see the effect in real time.

### Some limitations

Three things I'm not crazy about:

1. Changes to [external Markdown files](https://github.com/hakimel/reveal.js/#external-markdown) are not noticed by **live reload**.
2. The JavaScript that enables [stretching elements](https://github.com/hakimel/reveal.js/#stretching-elements) doesn't detect `class="stretch"` within Markdown.
3. The built-in [PDF export](https://github.com/hakimel/reveal.js/#pdf-export) doesn't attempt to be a [equivalent representation](http://lab.hakim.se/reveal-js?print-pdf) of the presentation.

So, I created a [small project](https://github.com/zharley/present) to sandbox some ideas and write a bit of scaffolding around the particular things that I wanted **Reveal.js** to do differently.

Try it out by cloning it:

{% highlight bash %}
git clone https://github.com/zharley/present
{% endhighlight %}

Install dependencies (among which is **Reveal.js** itself):

{% highlight bash %}
cd present
npm install
{% endhighlight %}

I also install [Decktape](https://github.com/astefanutti/decktape) for pixel-perfect export to PDF.

{% highlight bash %}
git clone --depth 1 https://github.com/astefanutti/decktape.git
curl -L http://astefanutti.github.io/decktape/downloads/phantomjs-osx-cocoa-x86-64 -o decktape/bin/phantomjs
chmod +x decktape/bin/phantomjs
{% endhighlight %}

To begin working on a new presentation, my script **new-dir** creates a directory containing a Markdown file `slides.md` and a subdirectory for assets (e.g. images).

{% highlight bash %}
./new-dir ~/test
{% endhighlight %}

Next, run the server command:

{% highlight bash %}
./serve ~/test
{% endhighlight %}

You should see this:

![Example slides.md](/assets/images/2015-12-03-222824-example-slides-md.png "Example slides.md")

To export to PDF, keep the server running and execute:

{% highlight bash %}
./export reveal-test.pdf
{% endhighlight %}

You should get [a PDF that looks just like the presentation](/assets/download/reveal-test.pdf).

### How it works

As the name suggests, my [Gruntfile.js](https://github.com/zharley/present/blob/master/Gruntfile.js) (based on **Reveal.js**'s) does most of the hard work. For example, following **watch** stanza enables live reload based on changes in the Markdown file (`config.slides`) or any assets (`config.assets`).

{% highlight js %}
watch: {
  options: {
    livereload: true
  },
  js: {
    files: [ 'Gruntfile.js', 'src/js/**/*.js' ],
    tasks: 'js'
  },
  theme: {
    files: [ 'src/css/theme/*.scss' ],
    tasks: 'sass'
  },
  html: {
    files: [ 'src/*.tpl', config.slides ],
    tasks: [ 'html', 'copy' ]
  },
  assets: {
    files: [ path.join(config.assets, '**') ],
    tasks: 'copy'
  },
},
{% endhighlight %}

The `index.html` file has been refactored as a template file [index.html.tpl](https://github.com/zharley/present/blob/master/src/index.html.tpl), in which contents are [injected dynamically](http://gruntjs.com/api/grunt.template).

{% highlight html+erb %}
<body>
  <div class="reveal">
    <!-- Any section element inside of this container is displayed as a slide -->
    <div class="slides">
      <%= slides %>
    </div>
  </div>
{% endhighlight %}

Finally, I modified the "stretching elements" functionality by writing a [custom plugin](https://github.com/zharley/present/blob/master/src/js/plugin/custom.js) using **Reveal.js**'s extension system. Now, stretchable images can be defined within Markdown.

### Next steps

There are many more **Reveal.js** features that I want to explore, including [MathJax integration](https://github.com/hakimel/reveal.js/#mathjax) (displaying math equations) and [multiplexing](https://github.com/hakimel/reveal.js/#multiplexing) (simultaneous presentation to multiple screens controlled from a single device).
