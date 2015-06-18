---
layout:     post
title:      "Selenium: Puppetry of the browser"
categories: automation screenshot webdriver
---
[Selenium](http://www.seleniumhq.org/) is an [automated testing tool](http://en.wikipedia.org/wiki/List_of_web_testing_tools) for performing [quality assurance](https://en.wikipedia.org/wiki/Quality_assurance) on websites. Basically, it launches a browser (like [Chrome](http://www.google.com/chrome/) or [Firefox](https://www.mozilla.org/en-US/firefox/) or [Safari](https://www.apple.com/ca/safari/)) and takes control of it, robotically visiting pages and clicking on things without human intervention. Let's explore how it works and why it's useful.

### Installing Selenium

Note that **Selenium** comes in two flavours: advanced and basic. We're only interested in the advanced one, called [Selenium Webdriver](http://www.seleniumhq.org/projects/webdriver/)  (not to be confused with the less powerful [Selenium IDE](http://www.seleniumhq.org/projects/ide/)).

To use **Selenium WebDriver**, you just need to write a few lines of code in one of [various supported languages](http://docs.seleniumhq.org/about/platforms.jsp). We'll use Ruby but [instructions are available for other languages](http://www.seleniumhq.org/docs/03_webdriver.jsp#setting-up-a-selenium-webdriver-project) as well. If you don't yet have Ruby installed, consider using [my rbenv installation method]({% post_url 2015-04-17-my-development-machine %}). Once Ruby is installed, you can just run:

{% highlight bash %}
gem install selenium-webdriver
{% endhighlight %}

### Becoming a browser puppeteer

A fun way to experiment with Selenium is to open an [interactive Ruby shell](http://ruby-doc.org/stdlib-2.0.0/libdoc/irb/rdoc/IRB.html). From a terminal, run:

{% highlight bash %}
irb
{% endhighlight %}

At the prompt, enter:

{% highlight ruby %}
require 'selenium-webdriver'
{% endhighlight %}

This loads the gem. You should get back a response like this:

> => true

Next, launch **Chrome**:

{% highlight ruby %}
driver = Selenium::WebDriver.for :chrome
{% endhighlight %}

You'll see a result like:

> => #<Selenium::WebDriver::Driver:0x..fdb8814522c6b86b4 browser=:chrome>

Meanwhile, an *actual browser window* should appear, as pictured below. It often launches behind other windows, so you may have to hunt for it with **Command+Tab** (or **Alt+Tab**).

[![Blank Selenium-created browser](/assets/images/2015-06-17-175049-blank-selenium-created-browser.png "Blank Selenium-created browser")](/assets/images/2015-06-17-175049-blank-selenium-created-browser.png)

Now, for the magic:

{% highlight ruby %}
driver.get "https://duckduckgo.com/"
{% endhighlight %}

And it's alive!

[![Launch DuckDuckGo](/assets/images/2015-06-17-175758-launch-duckduckgo.png "Launch DuckDuckGo")](/assets/images/2015-06-17-175758-launch-duckduckgo.png)

Try entering a search term:

{% highlight ruby %}
element = driver.find_element(:id, "search_form_input_homepage")
element.click
element.send_keys "selenium webdriver help"
{% endhighlight %}

Notice that the browser has actually filled in the search field! Now send the enter key: 

{% highlight ruby %}
element.send_keys :return
{% endhighlight %}

And you see the results!

[![DuckDuckGo search results](/assets/images/2015-06-17-180931-duckduckgo-search-results.png "DuckDuckGo search results")](/assets/images/2015-06-17-180931-duckduckgo-search-results.png)

Explore the [Ruby bindings reference](https://code.google.com/p/selenium/wiki/RubyBindings) to see what else you can do. When you're done, close the browser:

{% highlight ruby %}
driver.quit
{% endhighlight %}

### A practical example

Suppose you want to capture a screenshot of a given web page. The following script (called `take-screenshot`) accepts a URL as a parameter and uses Selenium to generate a screenshot image at a consistent size. For example:

{% highlight ruby %}
./take-screenshot "https://duckduckgo.com/" screenshot.png
{% endhighlight %}

By default, the script sets the browser size to **1024x768**, but this can be adjusted by specifying additional parameters. For example, a mobile view can be obtained like this:

{% highlight ruby %}
./take-screenshot "https://duckduckgo.com/" screenshot.png 320 480
{% endhighlight %}

The result is a **320x480** sized view:

[![Mobile DuckDuckGo](/assets/images/2015-06-17-182013-mobile-duckduckgo.png "Mobile DuckDuckGo")](/assets/images/2015-06-17-182013-mobile-duckduckgo.png)

Finally the script can also take a 5th parameter, if you would like to wait a number of seconds before taking the screenshot, in order to allow the page to fully render. See the full script:

{% highlight ruby %}
#!/usr/bin/env ruby
require 'selenium-webdriver'

# Check that parameters are given
if ARGV.size < 2
  puts "Usage: #{File.basename($0)} http://example.com/path/to/url /path/to/filename.png [width] [height] [sleep]"
  exit 1
end

# Extract parameters
url = ARGV[0]
filename = ARGV[1]
width = ARGV[2] || 1024
height = ARGV[3] || 768
seconds = ARGV[4].to_i

# File should not exist
if File.exist? filename
  raise "Destination file '#{filename}' already exists!"
end

# Generate screenshot
begin
  driver = Selenium::WebDriver.for :chrome
  driver.manage.window.resize_to width, height
  driver.get url
  sleep seconds
  driver.save_screenshot filename
ensure
  driver.quit
end

# File should now exist
if File.exist? filename
  puts "=> Screenshot saved: '#{filename}'"
else
  raise "Destination file '#{filename}' already exists!"
end
{% endhighlight %}

Note that the current version of this script (and others) can be found in [this project's Github repository](https://github.com/zharley/blog).

### Next steps

This illustrates how you can be become a browser puppeteer with little effort. Now identify some tedious web-based tasks and automate them! For quality assurance testing, you'll want to organize your tests into units, as described in [this post about Selenium, Ruby, Test::Unit and Rake](http://blog.tommymacwilliam.com/post/24147507987/selenium-test-unit-rake-painless-web). For general automation, just experiment and iterate. 
