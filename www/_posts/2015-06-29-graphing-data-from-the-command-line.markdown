---
layout:     post
title:      "Graphing data from the command line"
categories: gnuplot cli
---
When the need arises to plot some data, it's tempting to reach for the nearest bloated [spreadsheet program](https://en.wikipedia.org/wiki/List_of_spreadsheet_software). Instead, try [Gnuplot](http://www.gnuplot.info), a powerful, light-weight, command-line tool for generating graphs.

### Your first Gnuplot

I installed Gnuplot on [my OSX environment]({% post_url 2015-04-17-my-development-machine %}) with [Homebrew](http://brew.sh/):

{% highlight bash %}
brew install gnuplot
{% endhighlight %}

On [Ubuntu](http://www.ubuntu.com/) you can probably run **apt-get install gnuplot**. Otherwise, consult the [Gnuplot download page](http://www.gnuplot.info/download.html). Be sure to get a recent version.

{% highlight bash %}
gnuplot -V
{% endhighlight %}

> gnuplot 5.0 patchlevel 0

For starters, let's make a [sine](https://en.wikipedia.org/wiki/Sine) curve for values of **x** between -5 and +5, rendering the output as a [jpeg](https://en.wikipedia.org/wiki/JPEG).

{% highlight bash %}
gnuplot -e "set terminal jpeg; plot [-5:5] sin(x)" > out.jpeg
{% endhighlight %}

This generates an image called **out.jpeg**, shown below.

![A jpeg sine plot](/assets/images/2015-06-26-182434-a-jpeg-sine-plot.jpeg "A jpeg sine plot")

You can experiment with different output formats via the **set terminal** command. For example, try [svg](https://en.wikipedia.org/wiki/Scalable_Vector_Graphics) for a smoother-looking curve:

{% highlight bash %}
gnuplot -e "set terminal svg; plot [-5:5] sin(x)" > out.svg
{% endhighlight %}

![An svg sine plot](/assets/images/2015-06-26-183150-an-svg-sine-plot.svg "An svg sine plot")

List available output formats by running:

{% highlight bash %}
gnuplot -e "set terminal"
{% endhighlight %}

The options include:

* **canvas**: HTML Canvas object
* **cgm**: Computer Graphics Metafile
* **context**: ConTeXt with MetaFun (for PDF documents)
* **corel**: EPS format for CorelDRAW
* **dumb**: ascii art for anything that prints text
* **dxf**: dxf-file for AutoCad (default size 120x80)
* **eepic**: EEPIC -- extended LaTeX picture environment
* **emf**: Enhanced Metafile format
* **emtex**: LaTeX picture environment with emTeX specials
* **epslatex**: LaTeX picture environment using graphicx package
* **fig**: FIG graphics language for XFIG graphics editor
* **gif**: GIF images using libgd and TrueType fonts
* **hpgl**: HP7475 and relatives [number of pens] [eject]
* **jpeg**: JPEG images using libgd and TrueType fonts
* **latex**: LaTeX picture environment
* **lua**: Lua generic terminal driver
* **mf**: Metafont plotting standard
* **mp**: MetaPost plotting standard
* **pcl5**: HP Designjet 750C, HP Laserjet III/IV, etc. (many options)
* **png**: PNG images using libgd and TrueType fonts
* **postscript**: PostScript graphics, including EPSF embedded files (*.eps)
* **pslatex**: LaTeX picture environment with PostScript \specials
* **pstex**: plain TeX with PostScript \specials
* **pstricks**: LaTeX picture environment with PSTricks macros
* **qms**: QMS/QUIC Laser printer (also Talaris 1200 and others)
* **svg**: W3C Scalable Vector Graphics
* **tek40xx**: Tektronix 4010 and others; most TEK emulators
* **tek410x**: Tektronix 4106, 4107, 4109 and 420X terminals
* **texdraw**: LaTeX texdraw environment
* **tgif**: TGIF X11 [mode] [x,y] [dashed] ["font" [fontsize]]
* **tikz**: TeX TikZ graphics macros via the lua script driver
* **tkcanvas**: Tk/Tcl canvas widget [perltk] [interactive]
* **tpic**: TPIC -- LaTeX picture environment with tpic \specials
* **unknown**: Unknown terminal type - not a plotting device
* **vttek**: VT-like tek40xx terminal emulator
* **xterm**: Xterm Tektronix 4014 Mode

For fun, try **dumb**, which generates [ASCII art](https://en.wikipedia.org/wiki/ASCII_art).

{% highlight bash %}
gnuplot -e "set terminal dumb; plot [-5:5] sin(x)"
{% endhighlight %}

	    1 +-+----+-------------+-------------+------------+-------------+----+-+
	      |    +++             +             +      ++    ++            +      |
	  0.8 +-+   ++                                ++       ++   sin(x) +-----+-+
	      |       +                               +          +                 |
	  0.6 +-+     +                             ++            +              +-+
	      |        ++                           +              +               |
	  0.4 +-+       +                          +                +            +-+
	      |          +                        +                 +              |
	  0.2 +-+         +                       +                  +           +-+
	    0 +-+          +                     +                    +          +-+
	      |            +                    +                     +            |
	 -0.2 +-+           +                  +                       +         +-+
	      |              +                 +                        +          |
	 -0.4 +-+            +                +                          +       +-+
	      |               +              +                           ++        |
	 -0.6 +-+              +            ++                             +     +-+
	      |                 +          +                               +       |
	 -0.8 +-+                ++       ++                                ++   +-+
	      |      +            ++    ++       +            +             +++    |
	   -1 +-+----+-------------+-------------+------------+-------------+----+-+
		    -4            -2             0            2             4

### Plotting data from a file

For some sample data, let's use [historical browser usage stats](https://en.wikipedia.org/wiki/Usage_share_of_web_browsers) for [Firefox](https://www.mozilla.org/en-US/firefox/new/) on desktop as of January in the years indicated. Save it to a file called **firefox.tsv** (n.b. tsv = tab-separated values).

    2009    27.03
    2010    31.64
    2011    30.68
    2012    24.78
    2013    21.42
    2014    18.90
    2015    16.96

Plot to see Firefox's declining popularity.

{% highlight bash %}
gnuplot -e "set terminal svg; plot 'firefox.tsv'" > out.svg
{% endhighlight %}

![Firefox popularity point plot](/assets/images/2015-06-29-164616-firefox-popularity-point-plot.svg "Firefox popularity point plot")

Instead of points, a line is more informative:

{% highlight bash %}
gnuplot -e "set terminal svg; set style data lines; plot 'firefox.tsv'" > out.svg
{% endhighlight %}

![Firefox popularity line plot](/assets/images/2015-06-29-164702-firefox-popularity-line-plot.svg "Firefox popularity line plot")

Now try a bar graph, including a title and axis labels:

{% highlight bash %}
gnuplot -e "set terminal svg; set title 'Firefox popularity: 2009-2015'; set xlabel 'Year'; set ylabel '% usage'; set style fill solid 0.3; set style data boxes; plot 'firefox.tsv'" > out.svg
{% endhighlight %}

![Firefox popularity bar graph](/assets/images/2015-06-29-164829-firefox-popularity-bar-graph.svg "Firefox popularity bar graph")

### Writing plot recipes

As the previous example shows, **gnuplot -e** becomes unwieldy for non-trivial plots. Instead, we can execute the same commands from a file.

Write this to a new file called **firefox-usage.plot**.

{% highlight gnuplot %}
set terminal svg
set title 'Firefox popularity: 2009-2015'
set xlabel 'Year'
set ylabel '% usage'
set style fill solid 0.3
set style data boxes
plot 'firefox.tsv'
{% endhighlight %}

Run the commands using **gnuplot**.

{% highlight bash %}
gnuplot firefox-usage.plot > out.svg
{% endhighlight %}

The benefit of this approach becomes clear with a more complicated example. Save the following data to a new file called **usage.csv**.

	Year,Internet Explorer,Chrome,Firefox,Safari
	2009,65.41,1.38,27.03,2.57
	2010,55.25,6.04,31.64,3.76
	2011,46.00,15.68,30.68,5.09
	2012,37.45,28.40,24.78,6.62
	2013,30.71,36.52,21.42,8.29
	2014,22.85,43.67,18.90,9.73
	2015,19.28,48.15,16.96,10.28

Note that it contains a header row, is comma-delimited (rather than tab-delimited), and contains more interesting data.

Save the following plot commands as **usage.plot**.

{% highlight gnuplot %}
# Output W3C Scalable Vector Graphics
set terminal svg

# Read comma-delimited data from file
set datafile separator comma

# Set graph title
set title 'Browser popularity: 2009-2015'

# Set label of x-axis
set xlabel 'Year'

# Set label of y-axis
set ylabel '% usage'

# Use a line graph
set style data line

# Plot data from a file, with extra notes below:
#
# for [i=2:5]         Loop for values of i between 2 and 5 (inclusive)
# using i:xtic(1)     Plot column i using tick labels from column 1
# title columnheader  Use the column headers (first row) as titles
# linewidth 4         Use a wider line width
#
plot for [i=2:5] 'usage.csv' using i:xtic(1) title columnheader linewidth 4
{% endhighlight %}

Run **usage.plot**.

{% highlight bash %}
gnuplot usage.plot > out.svg
{% endhighlight %}

![Browser popularity line graph](/assets/images/2015-06-29-174652-browser-popularity-line-graph.svg "Browser popularity line graph")

### Creating a reusable plot script

With a few extra lines of code, a plot recipe can become a reusable script. Examine the script below and note that it:

* Includes a [shebang](https://en.wikipedia.org/wiki/Shebang_%28Unix%29) (`#!/usr/bin/env gnuplot -c`) to specify that it should be interpreted using **gnuplot** with **-c** to enable command-line arguments.
* Accepts an argument specifying the CSV file to plot.
* Validates, using `strlen(ARG1)`, that an argument was passed to the script.
* Refers to **ARG1** instead of a hard-coded filename in the **plot** command.

{% highlight gnuplot %}
#!/usr/bin/env gnuplot -c

# Show usage information if no argument is present
if (strlen(ARG1) == 0) print "Usage: " . ARG0 . " data.csv"; exit

# Output W3C Scalable Vector Graphics
set terminal svg

# Read comma-delimited data from file
set datafile separator comma

# Set graph title
set title 'Browser popularity'

# Set label of x-axis
set xlabel 'Year'

# Set label of y-axis
set ylabel '% usage'

# Use a histogram
set style data histogram

# Use clustered histogram (gap size of 1 makes xtics position better)
set style histogram clustered gap 1

# Use a solid fill style for histogram bars
set style fill solid 1 noborder

# Disable the tiny tick lines on the x-axis
set xtics scale 0

# Plot data from a file, with extra notes below:
#
# for [i=2:5]         Loop for values of i between 2 and 5 (inclusive)
# using i:xtic(1)     Plot column i using tick labels from column 1
# title columnheader  Use the column headers (first row) as titles
#
plot for [i=2:5] ARG1 using i:xtic(1) title columnheader
{% endhighlight %}

Save the above script as **plot-usage**, then make it executable:

{% highlight bash %}
chmod +x plot-usage
{% endhighlight %}

Finally, execute the script, supplying the **usage.csv** file as an argument.

{% highlight bash %}
./plot-usage usage.csv > out.svg
{% endhighlight %}

![Histogram produced by plot script](/assets/images/2015-06-29-230047-histogram-produced-by-plot-script.svg "Histogram produced by plot script")

Note that skipping the required argument simply produces usage information:

{% highlight bash %}
./plot-usage
{% endhighlight %}

> Usage: ./plot-usage data.csv

### Next steps

The waters are deep. Explore the [Gnuplot documentation](http://www.gnuplot.info/documentation.html) and [extensive demos](http://gnuplot.sourceforge.net/demo_svg_5.0/). Work on creating a toolkit of plot scripts that are relevant for you.
