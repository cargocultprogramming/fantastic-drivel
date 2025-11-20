---
layout: post
title: Manage ruby project environments
description: Manage ruby project environments with rbenv
tags: web-development ruby jekyll
categories: 
featured: false
---
## Summary

1. Install rbenv using <https://github.com/rbenv/rbenv-installer>
1. Activate it by running `~/.rbenv/rbenv init` and add to bash
1. Install local ruby version `rbenv install -l`
1. Set up in project directory `rbenv local x.x.x`
1. Install gems `bundle install` (check env with `gem env home`)
1. Optionally work with `rbenv-gemset` to manage gem versions too

## Details

For smaller websites I like to work with jekyll since it is nifty to use, flexible in development and deployment and also secure (because static). One issue with jekyll is however, that the underlying ruby support on ubuntu is not great. I repeatedly ran into issues with ruby, bundler and gems especially when working in collaboration with others.

These issues are pretty similar to what you get when working with python across different systems and in collaboration with other devs. Basically there is a broader issue where coding projects need to manage their dependencies on *per-project-basis* rather than traditionally relying on the dependency management of the system, the project is running on. In a nutshell: **Modern coding projects need to manage their dependencies *within* the project.**

Back to jekyll and ruby: When I started with jekyll, following the basic installation instructions, there was no mentioning of any management of ruby versions, which turned out to be a PITA, as mentioned above, where I had to spend hours sorting out the environment (for me as well as for collaborators) instead of working on the actual project. I always find this particularly frustrating as a hobby developer: you want to quickly update a simple website, but instead of 30 minutes for a quick update and push, you spend 2 days researching bugs and installing ruby / gems until a collaborator can even reproduce what already used to work. Recently however I stumbled across `rbenv` (<https://github.com/rbenv/rbenv>) which manages the ruby environment in a similar way as `poetry`[^1] does for python. `rbenv` seems to be the same game-changer that poetry is for python, when setting up environments. Though the project is ~10 years old, I never stumbled across it earlier - probably more of an issue with my research than anything else, but maybe also an indicator how slowly the old way of doing things (system dependency management) changes to new paradigms.

Anyway: The basic functionality: `rbenv` integrates with the shell and intercepts any ruby commands to manipulated the path for management. This works completely independent of the system ruby[^2] and allows the parallel installation of various ruby versions (and gems) plus seamless switching on project basis.

[^1]: `poetry` is a new-ish environment manages for python that manages python versions *and* packages for these versions for python coding projects. See: <https://python-poetry.org/>
[^2]: `rbenv` knows about the system-ruby so that you switch back to that if needed.

There are a couple of different ways to install. I'm not a fan of operating multiple package managers so I'm using the manual script installer from the side project <https://github.com/rbenv/rbenv-installer>. After the installation, I'm running the newly installed script `~/.rbenv/bin/rbenv init` which will just look for my shell and print out short instructions how to integrate the command. After this is done and active (`vim .bashrc` and `source`), it's time to install the first local ruby version (`rbenv install -l` then pick version). This copy is installed in userland, already independent of the system ruby.

There was a small hitch with building `psych` during this process, easily solved by installing `sudo apt install libyaml-dev` on ubuntu.

Finally to use rbenv in a project, I can then simply change to a project directory and run `rbenv local 3.3.0` to set the desired ruby version for that project. From then on, all calls to ruby will go the right version and for example `bundle install` will use a local `Gemfile` to install all necessary gems for that project in the local version of ruby. I can check if everything is correct by running `gem env home` which will print out the installation directory. If everything is fine, this will spit out something like `~/.rbenv/versions/3.3.0/lib...`, which is exactly what we want.

Note, that different to `poetry`, `rbenv` just switches the ruby version, but doesn't allow different versions of gems on a project basis. In order to do that theres another project called *rbenv-gemset* (<https://github.com/jf/rbenv-gemset>), which supports this.

Another option, which manages both ruby version management *and* gem management is `rvm` (<https://github.com/rvm/rvm>). It seems to be less popular than `rbenv` and I didn't test it yet.
