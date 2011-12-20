---
title: 'Using RVM with TorqueBox'
author: Lance Ball
layout: news
tags: [rvm, setup, installation]
---

[examples]: https://github.com/torquebox/torquebox/tree/master/integration-tests/apps/alacarte
[rvm]: http://rvm.beginrescueend.com/
[rvm-install]: http://rvm.beginrescueend.com/rvm/install/
[ci]: http://torquebox.org/torquebox-dev.zip
[gem-changes]: http://torquebox.org/news/2011/02/25/using-rvm-with-torquebox/
[the new article]: /news/2011/12/19/using-rvm-with-torquebox/

We've been getting quite a few requests in IRC lately asking how to set up and use [RVM][rvm] with TorqueBox.
In this post, we'll take a look at RVM, talk about why you might want to use it in your development
environment, and show you how to set it up for use with TorqueBox.

## Update - Tuesday, December 20, 2011
With the release of 2.0.0.beta1, the information in this article is obsolete for all but the 
1.x line.  For (simpler) instructions on 2.0.0.beta1, please see [the new article].

## Update - Friday, March 4, 2011
This article has been modified to reflect the changes in gem names recently [announced][gem-changes].

## What is RVM and Why Use It?
RVM is a command line tool that helps you keep track of multiple, named ruby environments.  Each
environment can have its own set of gems and has a specific ruby interpreter, such as 
ruby-1.8.7, ree-1.8.7, or jruby-1.5.6.  This makes it very simple to have separate, named gem sets
and ruby interpreters for each of your applications &#8212; isolating them from one another and
potential version conflicts.  This is useful in a number of circumstances:

  * You are working on a few different apps that have a large number of gem dependencies - some with
    conflicting versions, and you'd like to keep them isolated.
  * You have multiple environments for a single application (e.g. development, staging, production)
    and you want to ensure identical runtime environments on each.
  * You want an easy way to test upgrades to your gem dependencies.
  
For these and other reasons, I'm a big fan of RVM and use it in my daily development work.
If you're not already using RVM, you'll need to [install it][rvm-install] if you want to follow along.
Don't worry, the installation is quick and painless.

## Setting Up Your Development Environment
TorqueBox runs on some pretty awesome foundations like JBossAS and JRuby, both of which we ship with the
[continuous integration builds][ci]. With RVM, however, we'll install our own JRuby and use that. It's 
all the same to TorqueBox as long as we twist the right knobs.  First let's install JRuby under RVM &#8212;
TorqueBox currently requires 1.5.6, so that's what we'll install.

    $ rvm install jruby-1.5.6
    $ rvm use jruby-1.5.6
    $ rvm info
    
This will install the JRuby interpreter, switch your environment to the new install, and show you all
kinds of info about it. About half way down the `rvm info` output, you'll see the "homes" section.
Copy the ruby path, mine looks like this: `ruby: "/Users/lanceball/.rvm/rubies/jruby-1.5.6"`.
Set that as your JRUBY_HOME in your ~/.profile.

    echo "export JRUBY_HOME=/Users/lanceball/.rvm/rubies/jruby-1.5.6" >> ~/.profile

That's all you have to do to get TorqueBox to use RVM's JRuby interpreter &#8212; easy!

## Default Gemsets
Now we've got TorqueBox using RVM's JRuby interpreter, but to run apps under TorqueBox, you need
the TorqueBox gems installed.  So let's ensure that they are always available when we're using JRuby.
To get the latest TorqueBox gems, you'll need to make sure `gem` knows where to find them.  Open up
`~/.gemrc` and add rubygems.torquebox.org so that it looks like this:

    :sources:
    - http://rubygems.org
    - http://rubygems.torquebox.org
    
Then install the TorqueBox gems.

    $ rvm use jruby-1.5.6@global
    $ gem install torquebox torquebox-messaging-container torquebox-naming-container torquebox-capistrano-support torquebox-rake-support torquebox-vfs --pre 
    $ gem install bundler
    $ gem list
    
    *** LOCAL GEMS ***

    bundler (1.0.10)
    torquebox (1.0.0.CR1)
    torquebox-capistrano-support (1.0.0.CR1)
    torquebox-container-foundation (1.0.0.CR1)
    torquebox-messaging (1.0.0.CR1)
    torquebox-messaging-container (1.0.0.CR1)
    torquebox-naming (1.0.0.CR1)
    torquebox-naming-container (1.0.0.CR1)
    torquebox-rake-support (1.0.0.CR1)
    torquebox-vfs (1.0.0.CR1)

This installs all of the required TorqueBox gems into the global gemset for JRuby-1.5.6, ensuring their
availability whenever you're using JRuby &#8212; awesome!  I put bundler in there too, because I find it 
essential for all apps.  Be sure to use the `--pre` flag. We're working on the cutting edge here, and if
you're using a continuous integration build you'll want the latest pre-release gems.

## Application Gemsets
To keep separate gem sets for each application, use `rvm gemset`.  I use the app name for my gemset.

    $ rvm gemset create myapp
    $ rvm use jruby-1.5.6@myapp
    
Now you have a local GEM_PATH specific for your app, with the TorqueBox gems already available. To make
life even simpler, do this:

    $ cd myapp
    $ echo "rvm use jruby-1.5.6@myapp" > .rvmrc
    
Now whenever you change to your app directory, your JRuby interpreter and GEM_HOME are set.  Just add
your application gem dependencies to your Gemfile, run `bundle install` and you'll be in business.

## Staying Edgy
TorqueBox development is fast and furious. There are new builds every day with bug fixes, feature enhancements
and other goodies.  What happens when you want to download a new dev build and use these new features?
The `gem update` command will notice that you've already got the TorqueBox gems installed and do nothing.  
In this case, you'll need to uninstall and reinstall the gems.  

    $ rvm use jruby-1.5.6@global
    $ gem uninstall torquebox torquebox-messaging-container torquebox-naming-container torquebox-capistrano-support torquebox-rake-support torquebox-vfs 
    $ gem install torquebox torquebox-messaging-container torquebox-naming-container torquebox-capistrano-support torquebox-rake-support torquebox-vfs --pre 
    
While this is a bit of a pain, it's quicker and easier than re-installing all of your application gems
each time you want to update to a new TorqueBox build.

Good luck - and feel free to hit us up in IRC on #torquebox if you run into any questions.

