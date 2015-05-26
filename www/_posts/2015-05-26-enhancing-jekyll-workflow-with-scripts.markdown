---
layout:     post
title:      "Enhancing Jekyll workflow with scripts"
categories: bash ruby
---
After a few weeks of [using Jekyll]({% post_url 2015-04-23-creating-this-site-with-jekyll %}), I've developed a handful of simple scripts to improve my workflow. The code serves as an example of my *script everything* philosophy:

> If a task can be scripted, it usually should be.

As always, up-to-date versions of the source code can be found in [this project's Github repository](https://github.com/zharley/blog).

### Drafting new posts

When I begin a draft, I create a [Markdown](http://daringfireball.net/projects/markdown) file in the `_drafts` subdirectory named after the post title. For example, for a post that will be titled *A new post about programming*, I create a file called `a-new-post-about-programming.markdown`, which equates to an appropriate pretty URL once published.

The transformation from title to filename is pretty straightforward:

1. Make it lowercase
1. Convert spaces, quotes and other unwanted characters to dashes
1. Remove repeated dashes
1. Add the `.markdown` extension

Note that steps 1-3 are similar to [parameterize](http://apidock.com/rails/ActiveSupport/Inflector/parameterize), a method I had come across in [Ruby On Rails](http://rubyonrails.org/). In the Ruby script below, I adapt **parameterize** and create an appropriately named file in the `_drafts` subdirectory, given a title as a parameter.

{% highlight ruby %}
#!/usr/bin/env ruby

# @see http://apidock.com/rails/ActiveSupport/Inflector/parameterize
def parameterize(string, separator = '-')
  # Turn unwanted chars into the separator
  string = string.gsub(/[^A-Za-z0-9\-_]+/, separator)
  unless separator.nil? || separator.empty?
    re_separator = Regexp.escape(separator)
    # No more than one of the separator in a row.
    string.gsub!(/#{re_separator}{2,}/, separator)
    # Remove leading/trailing separator.
    string.gsub!(/^#{re_separator}|#{re_separator}$/, '')
  end
  string.downcase
end

# Check that a parameter (title) has been supplied
if ARGV.empty?
  puts "Usage: #{File.basename($0)} \"Some title for a post\""
  exit 1
end

# Combine all arguments, forgiving lack of quotes around title
title = ARGV.join(" ");

# Target directory for drafts
dir = "www/_drafts"

# Default format (and file extension)
format = "markdown"

# Build draft filename within target directory
path = File.join(dir, parameterize(title) + '.' + format)

# Check that target directory exists
unless Dir.exist?(dir)
  puts "=> Missing target directory #{dir}"
  exit 1
end

# Check that the draft filename does not already exist
if File.exist?(path)
  puts "=> File already exists #{path}"
  exit 1
end

# Prepare post metadata containing title
body = <<EOF
---
layout:     post
title:      "#{title}"
categories: 
---
EOF

# Write draft file
File.write(path, body)
puts "=> New draft post started at: #{path}"
{% endhighlight %}

For example, I can create a draft by running:

{% highlight bash %}
./new-draft "A new post about programming"
{% endhighlight %}

Optionally, I can omit quotes:

{% highlight bash %}
./new-draft A new post about programming
{% endhighlight %}

Either way, I get the following result:

> => New draft post started at: www/_drafts/a-new-post-about-programming.markdown

The generated file, `a-new-post-about-programming.markdown` is populated with some metadata (a.k.a. [front matter](http://jekyllrb.com/docs/frontmatter/) in the Jekyll vernacular).

{% highlight yaml %}
---
layout: post
title: "A new post about programming"
categories:
---
{% endhighlight %}

### Serving Jekyll locally

Jekyll supports previewing drafts by running:

{% highlight bash %}
jekyll serve --draft
{% endhighlight %}

I wrote a script to wrap this command, adding some color just for fun, and using port **4001** to avoid collisions with other Jekyll installations defaulting to port **4000**.

{% highlight bash %}
#! /bin/bash
set -e

# Custom port
MY_PORT=4001

# Some colours
MY_COLOR_NORMAL='\033[0m'
MY_COLOR_YELLOW='\033[0;33m'
MY_COLOR_BOLD_YELLOW='\033[1;33m'
MY_COLOR_UNDERLINE_CYAN='\033[4;36m'

# Show drafts by default (when no parameter is given)
if [ -z "$1" ]; then
    MY_DRAFTS=" --draft"
fi

echo -ne "=> ${MY_COLOR_YELLOW}Serving Jekyll locally"
if [ -n "$MY_DRAFTS" ]; then
    echo -ne " $MY_COLOR_BOLD_YELLOW(showing drafts)"
fi
echo -e "$MY_COLOR_NORMAL"

echo -e "=> See: ${MY_COLOR_UNDERLINE_CYAN}http://127.0.0.1:$MY_PORT/$MY_COLOR_NORMAL"

cd www

jekyll serve --port $MY_PORT$MY_DRAFTS
{% endhighlight %}

Therefore, to run Jekyll with drafts on my preferred port, I run:

{% highlight bash %}
./serve
{% endhighlight %}

The output looks something like this:

![Colored console output](/assets/images/2015-05-22-165615-colored-console-output.png "Colored console output")

Note that, as a convenience, supplying any parameter to the **serve** script will turn off drafts.

{% highlight bash %}
./serve 0
{% endhighlight %}

### Promoting drafts to published posts

Finally, I need to move my draft file from `_drafts` to `_posts`, prefixing it with the current date in `YYYY-MM-DD` format. This can be accomplished manually, of course, but the following script makes it foolproof.

{% highlight ruby %}
#!/usr/bin/env ruby

def get_pick(count)
  prompt = "=> Pick one or hit ^C to cancel [1-#{count}]: "
  pick = nil

  while pick.nil? do
    print prompt
    pick = STDIN.gets.chomp.to_i
    pick = nil if pick < 1 or pick > count
  end

  pick
end

drafts = Dir["www/_drafts/*"]
dir = "www/_posts"

# List drafts
puts "=> Post which draft?"
count = 0
for draft in drafts
  count += 1
  puts "%2d. %s" % [ count, draft ]
end

# Get selection
pick ||= get_pick(count)
pick = pick.to_i
raise "Invalid pick" if pick < 1 || pick > count
draft = drafts[pick - 1]

# Generate new filename
datestamp = Time.now.strftime("%Y-%m-%d")
filename = datestamp + "-" + File.split(draft).last
post = File.join(dir, filename)

# Move the file to its new location
system "mv #{draft} #{post}"
puts "=> Moved #{draft} to #{post}"
{% endhighlight %}

As a result, I can run:

{% highlight bash %}
./post-draft
{% endhighlight %}

And select which draft to publish:

    => Post which draft?
     1. www/_drafts/a-new-post-about-programming.markdown
     2. www/_drafts/enhancing-jekyll-workflow-with-scripts.markdown
    => Pick one or hit ^C to cancel [1-2]:

Pressing **1** and then **Enter** makes it happen.

### Summary

I've presented 3 scripts, consisting of 78 total lines of code, that make 3 routine Jekyll tasks marginally easier. Hopefully, over the lifetime of the project, this effort will produce a net savings of time. Either way, it represents a strategy of proactive automation (i.e. *script everything*) and forms the foundation for future enhancements.
