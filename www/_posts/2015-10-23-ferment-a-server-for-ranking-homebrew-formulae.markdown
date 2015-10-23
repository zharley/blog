---
layout:     post
title:      "Ferment: A server for ranking Homebrew formulae"
categories: vagrant ruby react
---
As a follow-up to [an earlier post about ranking Homebrew formulae]({% post_url 2015-08-28-osx-software-discovery-via-homebrew %}), this post describes creating a web page to show formulae by rank, with links to more information. The intent of this project, dubbed **Ferment**, is to enable discovery of new and interesting software that may not otherwise be noticed. The source [code is on GitHub](https://github.com/zharley/ferment) and (so far) looks like this in a browser:

[![Ferment running in a Vagrant box](/assets/images/2015-10-23-072107-ferment-running-in-a-vagrant-box.png "Ferment running in a Vagrant box")](/assets/images/2015-10-23-072107-ferment-running-in-a-vagrant-box.png)

In a nutshell, we analyze [Homebrew's GitHub repo](https://github.com/Homebrew/homebrew) and extract metadata from its catalog of software (a.k.a. formulae). The "first pass" takes a long time (several minutes) but subsequent updates are faster because the metadata is cached in a [MongoDB](https://www.mongodb.org/) database. To encapsulate the dependencies, we use [Vagrant](https://www.vagrantup.com/) and define a **Vagrantfile**.

### Running Ferment with Vagrant

Install [VirtualBox](https://www.virtualbox.org/), [Vagrant](https://www.vagrantup.com/) and a Vagrant plugin called [hostmanager](https://github.com/smdahlen/vagrant-hostmanager). Then, simply launch the VM.

{% highlight bash %}
vagrant up
{% endhighlight %}

Upon `vagrant up`, **Vagrant** reads the [Vagrantfile](https://github.com/zharley/ferment/blob/master/Vagrantfile) and creates a [Ubuntu 14.04 LTS](http://releases.ubuntu.com/trusty/) server, using the **hostmanager** plugin to assign a custom host name name (e.g. **ferment.example.com**).

Finally, **Vagrantfile** points to a [provision script](https://github.com/zharley/ferment/blob/master/bin/provision), which installs extra packages and performs configuration. This includes adding an hourly cron job to keep rankings up-to-date. In the provision script (and hourly via cron), the application calls **gulp**, which refers to [a JavaScript build system called Gulp](http://gulpjs.com/).

### The Gulp build script

The heart of the application's activity occurs in [gulpfile.js](https://github.com/zharley/ferment/blob/master/gulpfile.js), copied below for reference.

{% highlight javascript %}
var gulp = require('gulp');
var react = require('gulp-react');
var git = require('gulp-git');
var fs = require('fs');
var shell = require('gulp-shell')

gulp.task('brew', function () {
  if (fs.existsSync('homebrew')) {
    git.pull('origin', 'master', { cwd: 'homebrew' }, function (err) {
      if (err) throw err;
    });
  } else {
    git.clone('https://github.com/Homebrew/homebrew', function (err) {
      if (err) throw err;
    });
  }
});

gulp.task('collect', shell.task([
  './bin/collect homebrew'
]));

gulp.task('info', shell.task([
  './bin/info homebrew'
]));

gulp.task('rank', shell.task([
  './bin/rank'
]));

gulp.task('dump', shell.task([
  './bin/dump > dist/data.json'
]));

gulp.task('copy', function () {
  return gulp.src('src/index.html')
    .pipe(gulp.dest('dist'))
})

gulp.task('react', function () {
  return gulp.src('src/index.js')
    .pipe(react())
    .pipe(gulp.dest('dist'));
})
 
gulp.task('default', [ 'brew', 'collect', 'info', 'rank', 'dump', 'copy', 'react' ], function () {
});
{% endhighlight %}

The **default** task executes all required tasks in sequence. Once complete, navigate to [ferment.example.com](http://ferment.example.com/) to see the result.

### Next steps

This iteration of the **Ferment** server carries some unnecessary baggage (e.g. Ruby) and the frontend is just my experimental trial of [React](https://facebook.github.io/react/). To be continued.
