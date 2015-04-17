---
layout:     post
title:      "My development machine"
categories: osx yosemite setup
---
The following are my steps for configuring a new development machine running **OSX Yosemite**.

1. [Install Xcode](https://developer.apple.com/xcode/downloads/), which comes with essential build tools for OSX and iOS as well as a sweet [mobile device emulator](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/iOS_Simulator_Guide/). After installation, try printing the path of the xcode developer directory:

       xcode-select -p

    You should see something like `/Applications/Xcode.app/Contents/Developer` as output. If this does not work, try running:
    
       xcode-select --install

1. Install [Homebrew](http://brew.sh/), also known as the *missing package manager for OSX*. It's a program similar to [apt-get](http://en.wikipedia.org/wiki/Advanced_Packaging_Tool) for [Debian](https://www.debian.org/) and [Ubuntu](http://www.ubuntu.com/) that makes installing common software dead simple.

       ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

    Confirm that this worked by checking the version number (e.g. `Homebrew 0.9.5`):

       brew -v

1. In OSX, the default terminal program called **Terminal** is passable, but [iTerm2 has way more features](http://iterm2.com/features.html). Download it from the [iTerm2 home page](http://iterm2.com/) and enjoy.

1. Install a "version manager" for [Ruby](http://www.ruby-lang.org/) and [Rubygems](http://rubygems.org/), which allows you to switch between older and newer versions as required. Both [RVM](https://rvm.io/rvm/install) and [rbenv](https://github.com/sstephenson/rbenv) are great for this purpose, but I tend to agree with the [reasons for rbenv](https://github.com/sstephenson/rbenv/wiki/Why-rbenv%3F) given by its author.

    * To install **rbenv**:

          brew install rbenv

    * Carefully follow the post-installation instructions, and in particular, the step that involves adding `eval "$(rbenv init -)"` to your profile script (usuall either .bashrc or .zshrc). Confirm that this worked by checking the version number (e.g. `rbenv 0.4.0`) in a **new terminal window**.

          rbenv -v

    * Install the [ruby-build](https://github.com/sstephenson/ruby-build) plugin, which simplifies installing new Ruby versions.
	
          brew install ruby-build

    * Install a recent version of Ruby to use globally.

          rbenv install 2.2.1
          rbenv global 2.2.1

    * Open a new terminal window, and test that the desired version of Ruby is being used (should say something like `ruby 2.2.1`).

          ruby -v

1. For Java work (including Android below) [download the Java Development Kit](http://www.oracle.com/technetwork/java/javase/downloads/index.html). Once installed, confirm it by checking the version number (e.g. `javac 1.8.0_40`).

       javac -version

1. For Android development, also [install Android Studio](http://developer.android.com/sdk/index.html). Note that [the instructions](http://developer.android.com/sdk/installing/adding-packages.html) say:

    > By default, the Android SDK does not include everything you need to start developing.

    Therefore, follow up with the [Adding SDK Packages](http://developer.android.com/sdk/installing/adding-packages.html) section.

1. If you want to rename your machine, use `sudo` to [act as a superuser](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man8/sudo.8.html) and run `scutil`, a command that [manages system configuration](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man8/scutil.8.html). Replace "example" with your actual new host (i.e. computer) name.

       sudo scutil --set HostName example 

That's all for now. By the way, I'm currently using a *MacBook Pro Retina, 15-inch, Early 2013, 2.4 GHz Intel Core i7, 16 GB 1600 MHz DDR3*.

