---
layout:     post
title:      "OSX software discovery via Homebrew"
categories: osx homebrew git
---
Many operating systems have a [package manager](https://en.wikipedia.org/wiki/Package_manager), which facilitates the installation of software and associated dependencies. This post describes exploiting [Homebrew](http://brew.sh/), an OSX package manager, to discover interesting software by means of [Hacker News](https://news.ycombinator.com/)-inspired popularity ranking.

### How Homebrew works

First, [get Homebrew](http://brew.sh/). I've [already installed it]({% post_url 2015-04-17-my-development-machine %}), and this is my version number:

{% highlight bash %}
brew -v
{% endhighlight %}

> Homebrew 0.9.5 (git revision ddda; last commit 2015-08-26)

For demonstration purposes, let's install a clever little program called [tree](http://mama.indstate.edu/users/ice/tree/), which generates a graphical representation of a directory and subdirectories.

{% highlight bash %}
brew install tree
{% endhighlight %}

> ==> Downloading http://mama.indstate.edu/users/ice/tree/src/tree-1.7.0.tgz
> ######################################################################## 100.0%
> ==> make prefix=/usr/local/Cellar/tree/1.7.0 MANDIR=/usr/local/Cellar/tree/1.7.0/share/man/man1 CC
> 🍺  /usr/local/Cellar/tree/1.7.0: 7 files, 128K, built in 3 seconds

Try it out by creating a few test directories and running **tree**:

{% highlight bash %}
mkdir -p test/{a,b,c,d}/{d,e}
touch test/a/foo test/d/bar
tree test
{% endhighlight %}

This should give the result:

	test
	├── a
	│   ├── d
	│   ├── e
	│   └── foo
	├── b
	│   ├── d
	│   └── e
	├── c
	│   ├── d
	│   └── e
	└── d
	    ├── bar
	    ├── d
	    └── e

To see how **homebrew** knew how to install **tree**, call **brew info**:

{% highlight bash %}
brew info tree
{% endhighlight %}

> tree: stable 1.7.0
> Display directories as trees (with optional color/HTML output)
> http://mama.indstate.edu/users/ice/tree/
> /usr/local/Cellar/tree/1.7.0 (7 files, 128K) *
>   Built from source
> From: https://github.com/Homebrew/homebrew/blob/master/Library/Formula/tree.rb

Notice that it lists a [GitHub](https://github.com/) URL. Visiting `tree.rb` reveals a recipe for downloading and building the software.

[![Homebrew formula for tree](/assets/images/2015-08-13-172926-homebrew-formula-for-tree.png "Homebrew formula for tree")](/assets/images/2015-08-13-172926-homebrew-formula-for-tree.png)

In fact, *every* Homebrew package has a corresponding formula in the Homebrew git repo. To explore further, let's clone it:

{% highlight bash %}
git clone https://github.com/Homebrew/homebrew
{% endhighlight %}

Next, examine the change log for that particular formula:

{% highlight bash %}
cd homebrew
git log --pretty=format:'%cr - %s' Library/Formula/tree.rb
{% endhighlight %}

We can see that the formula was updated 25 times since it was added about 6 years ago.

* 3 weeks ago - Formula files style updates.
* 3 months ago - Add descriptions to all remaining homebrew packages
* 8 months ago - tree: switch mirror
* 11 months ago - Remove debugging code again
* 11 months ago - Delay requiring irb until runtime
* 12 months ago - Remove debugging code :shame:
* 12 months ago - Ensure log file is closed
* 12 months ago - tree: add mirror
* 1 year, 4 months ago - Fix tree
* 1 year, 4 months ago - tree: update 1.7.0 bottle.
* 1 year, 4 months ago - tree 1.7.0
* 1 year, 6 months ago - tree: add bottle.
* 1 year, 7 months ago - tree: a test
* 1 year, 9 months ago - Remove uses of -no-cpp-precomp
* 3 years ago - Batch convert MD5 formula to SHA1.
* 3 years, 7 months ago - tree: avoid inreplace
* 4 years, 1 month ago - tree 1.6.0
* 4 years, 4 months ago - tree: update download url
* 4 years, 6 months ago - Use ruby style for inheritance.
* 5 years ago - Update formulae to use ENV.cflags
* 5 years ago - Update formulae for version 0.7
* 5 years ago - Update tree to 1.5.3
* 6 years ago - s/require 'brewkit'/require 'formula'/g
* 6 years ago - ENV.cc; returns the compiler we use
* 6 years ago - Don't hardcode 'gcc' in manual formulas.
* 6 years ago - Tree formula

Let's compare this to the **git** formula. The following command just counts the commits rather than listing them all.

{% highlight bash %}
git rev-list HEAD --count Library/Formula/git.rb
{% endhighlight %}

> 292

As may be expected, the popular **git** formula has had 10X more updates than **tree**.

### Collecting formula meta-data

Let's see how many formulae there are right now:

{% highlight bash %}
ls -1 Library/Formula/*.rb | wc -l
{% endhighlight %}

> 3195

For each formula, the commit history contains two key data points:

* **Number of commits**: a possible indication of hotness
* **Date added to homebrew**: a possible indication of newness

After some fiddling with shell scripts, I settled on a Ruby script that collects the above data in a local [mongoDB](https://www.mongodb.org) database. This requires the relevant brew package and a [Ruby gem](https://rubygems.org/).

{% highlight bash %}
brew install mongodb
gem install mongo
{% endhighlight %}

Refer to the [mongo gem documentation](https://github.com/mongodb/mongo-ruby-driver) on how Ruby is used to control Mongo. The following script is called **collect**:

{% highlight ruby %}
#!/usr/bin/env ruby
require 'shellwords'
require 'mongo'

# Change modes (kinds of changes that can happen to a git-tracked file)
MODE_ADD    = 'A'
MODE_MODIFY = 'M'
MODE_DELETE = 'D'

# This script requires an argument
if ARGV.empty?
  puts "Usage: #{File.basename($0)} /path/to/homebrew"
  exit 1
end

# Connect to mongo
mongo = Mongo::Client.new('mongodb://127.0.0.1:27017/ferment')
Mongo::Logger.logger       = ::Logger.new('mongo.log')
Mongo::Logger.logger.level = ::Logger::INFO

# Enter Homebrew git directory
dir = ARGV[0]
Dir.chdir(dir)

# Pull latest changes from Git
#system "git pull"

# Create an index on revisions
mongo[:revisions].indexes.create_one({ :hash => 1 }, :unique => true)

# Get all revisions
hashes = `git rev-list HEAD`.split(/\s+/).reverse

# Report number of hashes found
puts "Found #{hashes.size} hashes."
count = 0

# Iterate through hashes of all commits
for hash in hashes
  # Skip this hash if we've seen it before
  next if mongo[:revisions].find(hash: hash).count > 0

  # Extract detailed changes for this commit
  changes = `git diff-tree --no-commit-id --name-status -r #{hash.shellescape}`.split("\n")

  # Extract unix timestamp
  timestamp = `git show -s --format='%at' #{hash.shellescape}`.chomp.to_i

  for change in changes
    # Extract mode and name from change string
    # e.g. "M	Library/Formula/harfbuzz.rb"
    if change =~ /([A-Z]+)\s+Library\/Formula\/(\w+).rb/
      # Mode is "M" in the example above
      mode = $1

      # Name is "harfbuzz" in the example above
      name = $2

      if mode == MODE_ADD
        # Insert a new formula
        mongo[:formulae].insert_one({
          name: name,
          count: 1,
          added: timestamp,
          score: 0
        })
      elsif mode == MODE_DELETE
        # Delete a formula
        mongo[:formulae].find(:name => name).find_one_and_delete
      elsif mode == MODE_MODIFY
        # Modify a formula
        mongo[:formulae].find(:name => name).update_one("$inc" => { count: 1 })
      else
        puts "Unsupported mode '#{mode}' for formula '#{name}'"
        next
      end
    end
  end

  # Store this revision
  revision = {
    hash: hash,
    changes: changes,
    timestamp: timestamp
  }

  # This revision has been digested
  mongo[:revisions].insert_one(revision)
end
{% endhighlight %}

You can execute it by supplying, as a parameter, a path to the cloned **Homebrew** repo:

{% highlight bash %}
./collect ~/src/homebrew/
{% endhighlight %}

After a few minutes, this captures all desired meta-data on formulae. At any time, you can see what it's doing by running:

{% highlight bash %}
mongoexport -h 127.0.0.1 -d ferment -c formulae --limit=10
{% endhighlight %}

You can see some stats like `count` which represents the number of commits and `added` which represents the UNIX timestamp of when the formula was added to Homebrew.

	{"_id":{"$oid":"55df987c89bbee0775000045"},"added":1244139679,"count":22,"name":"ack","score":0}
	{"_id":{"$oid":"55df987c89bbee0775000046"},"added":1244139679,"count":14,"name":"asciidoc","score":0}
	{"_id":{"$oid":"55df987c89bbee0775000047"},"added":1244139679,"count":36,"name":"boost","score":0}
	{"_id":{"$oid":"55df987c89bbee0775000048"},"added":1244139679,"count":30,"name":"cmake","score":0}
	{"_id":{"$oid":"55df987c89bbee0775000049"},"added":1244139679,"count":21,"name":"dmd","score":0}
	{"_id":{"$oid":"55df987c89bbee077500004a"},"added":1244139679,"count":10,"name":"fftw","score":0}
	{"_id":{"$oid":"55df987c89bbee077500004b"},"added":1244139679,"count":99,"name":"git","score":0}
	{"_id":{"$oid":"55df987c89bbee077500004c"},"added":1244139679,"count":12,"name":"grc","score":0}
	{"_id":{"$oid":"55df987c89bbee077500004d"},"added":1244139679,"count":11,"name":"lame","score":0}
	{"_id":{"$oid":"55df987c89bbee077500004e"},"added":1244139679,"count":11,"name":"liblastfm","score":0}
	2015-08-27T19:12:18.905-0400	exported 10 records

### Ranking formulae by newness and hotness

According to [a thread on Hacker News](https://news.ycombinator.com/item?id=231209), their [home page](https://news.ycombinator.com/) ranking calculation is simply:

	(p - 1) / (t + 2)^1.5

...where **p** is "points awarded" and **t** is "age of the post" in hours. In other words, the ranking linearly increases with points awarded and decays exponentially as it ages in hours.

The following script, called **rank**, does something similar with **homebrew** formulae, but counts a commit as a point and decays in months (rather than hours).

{% highlight ruby %}
#!/usr/bin/env ruby
require 'shellwords'
require 'mongo'

# Connect to mongo
mongo = Mongo::Client.new('mongodb://127.0.0.1:27017/ferment')
Mongo::Logger.logger       = ::Logger.new('mongo.log')
Mongo::Logger.logger.level = ::Logger::INFO

# Create an index on the formula's score
mongo[:formulae].indexes.create_one({ :score => 1 })

# Examine all formulae
formulae = mongo[:formulae]

now = Time.now.to_i
day = 60 * 60 * 24
month = day * 30

formulae.find.each do |formula|
  # Age in seconds
  age = now - formula["added"]

  # Calculate age in months
  age = age / month.to_f
  
  score = (formula["count"] - 1) / ((age + 2) ** 1.5)

  formulae.find(:_id => formula["_id"]).update_one("$set" => { :score => score })
end

formulae = mongo[:formulae].find.sort(:score => -1).limit(100)

formulae.find.each do |formula|
  age = (now - formula["added"]) / day
  normalized_score = (formula['score'] * 1000).to_i
  puts "* #{formula['name']} (days_old=#{age}, commits=#{formula['count']}, score=#{normalized_score})"
end
{% endhighlight %}

Subjectively, using "months" as the measure of a forumla's age seemed right. I experimented to find a time interval that maximized decay speed without losing stalwarts like **vim**, **node** and **git** in the top 100.

{% highlight bash %}
./rank
{% endhighlight %}

And here's the result:

* iojs (days_old=215, commits=58, score=2052)
* thefuck (days_old=127, commits=32, score=1980)
* planck (days_old=24, commits=10, score=1914)
* syncthing (days_old=443, commits=116, score=1674)
* embulk (days_old=117, commits=18, score=1179)
* tutum (days_old=311, commits=51, score=1147)
* pushpin (days_old=182, commits=27, score=1129)
* flow (days_old=282, commits=44, score=1116)
* carthage (days_old=251, commits=38, score=1107)
* influxdb (days_old=660, commits=116, score=977)
* mycli (days_old=32, commits=6, score=930)
* ford (days_old=56, commits=8, score=918)
* osquery (days_old=302, commits=39, score=905)
* awscli (days_old=538, commits=80, score=887)
* telegraf (days_old=70, commits=9, score=880)
* nghttp2 (days_old=198, commits=23, score=867)
* fpp (days_old=116, commits=13, score=843)
* packer (days_old=106, commits=12, score=838)
* h2o (days_old=226, commits=25, score=811)
* ansible (days_old=724, commits=102, score=755)
* docker (days_old=566, commits=72, score=744)
* commonmark (days_old=229, commits=23, score=735)
* ponyc (days_old=104, commits=10, score=700)
* creduce (days_old=130, commits=12, score=689)
* gcc (days_old=499, commits=56, score=682)
* sslmate (days_old=304, commits=29, score=662)
* bitrise (days_old=22, commits=4, score=659)
* passenger (days_old=798, commits=100, score=646)
* vegeta (days_old=199, commits=17, score=630)
* zurl (days_old=190, commits=16, score=620)
* hayai (days_old=44, commits=5, score=619)
* libressl (days_old=412, commits=39, score=608)
* emscripten (days_old=499, commits=50, score=608)
* terraform (days_old=353, commits=32, score=606)
* galen (days_old=217, commits=18, score=604)
* norm (days_old=64, commits=6, score=594)
* boot2docker (days_old=564, commits=57, score=589)
* gauge (days_old=317, commits=27, score=582)
* fleetctl (days_old=477, commits=45, score=580)
* softhsm (days_old=83, commits=7, score=571)
* vim (days_old=1041, commits=125, score=557)
* juju (days_old=756, commits=80, score=556)
* duck (days_old=233, commits=18, score=555)
* anjuta (days_old=32, commits=4, score=552)
* fig (days_old=452, commits=39, score=537)
* agda (days_old=218, commits=16, score=529)
* gdl (days_old=35, commits=4, score=527)
* dcd (days_old=153, commits=11, score=526)
* rethinkdb (days_old=941, commits=99, score=508)
* rem (days_old=14, commits=3, score=507)
* node (days_old=2147, commits=318, score=502)
* ipfs (days_old=62, commits=5, score=485)
* qt5 (days_old=979, commits=100, score=485)
* cryptol (days_old=150, commits=10, score=483)
* swiftlint (days_old=101, commits=7, score=479)
* algernon (days_old=83, commits=6, score=478)
* skinny (days_old=313, commits=22, score=478)
* ghq (days_old=85, commits=6, score=466)
* xplanetfx (days_old=486, commits=37, score=462)
* exercism (days_old=44, commits=4, score=461)
* jenkins (days_old=1678, commits=204, score=460)
* scriptcs (days_old=157, commits=10, score=459)
* minisign (days_old=44, commits=4, score=458)
* folly (days_old=67, commits=5, score=457)
* allegro (days_old=87, commits=6, score=456)
* mockserver (days_old=109, commits=7, score=448)
* sysdig (days_old=511, commits=38, score=445)
* xhyve (days_old=71, commits=5, score=435)
* gtkextra (days_old=229, commits=14, score=434)
* pcap_dnsproxy (days_old=72, commits=5, score=431)
* git (days_old=2275, commits=292, score=423)
* cig (days_old=116, commits=7, score=421)
* libbpg (days_old=264, commits=16, score=420)
* stdman (days_old=51, commits=4, score=419)
* pandoc (days_old=515, commits=36, score=416)
* fzf (days_old=537, commits=38, score=415)
* vault (days_old=52, commits=4, score=414)
* keybase (days_old=343, commits=21, score=405)
* nvm (days_old=633, commits=46, score=405)
* gexiv2 (days_old=159, commits=9, score=404)
* peco (days_old=211, commits=12, score=404)
* oauth2_proxy (days_old=27, commits=3, score=400)
* geographiclib (days_old=376, commits=23, score=396)
* cless (days_old=162, commits=9, score=394)
* arangodb (days_old=1202, commits=107, score=388)
* scw (days_old=57, commits=4, score=388)
* python (days_old=2219, commits=258, score=387)
* khal (days_old=58, commits=4, score=381)
* extract_url (days_old=58, commits=4, score=381)
* pypy3 (days_old=431, commits=26, score=377)
* pazpar2 (days_old=531, commits=34, score=377)
* deis (days_old=259, commits=14, score=374)
* graphite2 (days_old=61, commits=4, score=368)
* python3 (days_old=1958, commits=204, score=367)
* wellington (days_old=247, commits=13, score=365)
* saltstack (days_old=661, commits=44, score=364)
* gedit (days_old=64, commits=4, score=354)
* pla (days_old=91, commits=5, score=352)
* purescript (days_old=183, commits=9, score=345)
* mono (days_old=531, commits=31, score=342)

Maybe this needs a web page. For now, [the code is on GitHub](https://github.com/zharley/ferment).
