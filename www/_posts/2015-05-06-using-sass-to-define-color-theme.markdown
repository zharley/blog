---
layout:     post
title:      "Using SASS to define a color theme"
categories: jekyll sass
---
Since [creating this site with Jekyll]({% post_url 2015-04-23-creating-this-site-with-jekyll %}), I've endured the default color theme. Now I want to try something a little different. According to my friend and design guru [Steve](https://twitter.com/stephen_sills), a good way to get started is to pick a couple of [complementary colors](http://en.wikipedia.org/wiki/Complementary_colors).

So I read a little about [color theory](http://www.colormatters.com/color-and-design/basic-color-theory), the idea that interesting color relationships are predicted mathematically. Complementarity is one such relationship, which describes pairs of colors that *cancel eachother out*. For paint pigments and websites, this refers to pairs of contrasting colours that produce black when mixed together (n.b. complementary colors of *light* cancel to produce white instead). This is illustrated strikingly by [Johann Wolfgang von Goethe's](http://en.wikipedia.org/wiki/Johann_Wolfgang_von_Goethe) color wheel (below), which arranges colors so that complementary pairs are 180 degrees apart. There are [other color wheels](http://en.wikipedia.org/wiki/Color_wheel), but I like this one.

![Goethe's Color Wheel](/assets/images/2015-04-27-215028-goethe-s-color-wheel.png "Goethe's Color Wheel")

A newer color wheel can be found on [Adobe Color CC](https://color.adobe.com/), a great tool which lets you explore infinite combinations of related colors using different color rules. Using the complementary color rule, for example, you can pick 5 saturation points along any axis. I arrived at the following palette because it reminds me of a sandy beach. 
 
![Complementary colors](/assets/images/2015-04-27-214013-complementary-colors.png "Complementary colors")

To apply my colors site-wide, I edited the [Syntactically Awesome Stylesheets](http://sass-lang.com/) (or Sass) supplied by Jekyll. Sass is CSS language that that allows you to write [drier](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself) stylesheets by, among other things, defining variables. As a side note, the original Sass syntax (which used files named `.sass`) has evolved into "Sassy CSS" (which uses the `.scss` extension). There are interesting [reasons for Sass' evolving syntax](http://thesassway.com/editorial/sass-vs-scss-which-syntax-is-better), but in any case, Jekyll uses SCSS.

So, I copied my 5 new colors into `css/main.scss` like this:

{% highlight scss %}
$color-a: #9E8C6D;
$color-b: #FDFDFD;
$color-c: #FFF2DD;
$color-d: #1D77D4;
$color-e: #C9E4FF;
{% endhighlight %}

Then I sprinkled references to my new colors into `_sass/_base.scss` and `_sass/_layout.scss`, producing the effect below:

[![My own site with new palette](/assets/images/2015-05-06-180204-my-own-site-with-new-palette.png "My own site with new pallette")](/assets/images/2015-05-06-180204-my-own-site-with-new-palette.png)

Okay, it's not spectacular, but it's my *minimum viable palette*.
