---
layout: post
title: Configure a VPS for web apps
description: A minimal setup to run a web app on a vps
tags: linux cli python
categories: 
featured: true
toc:
    sidebar: right
---
A VPS (*V*irtual *P*rivate *S*erver) is a cheap way to run web apps for hobby projects. Basically you are renting a small virtual server, that will not have any form of guaranteed performance (since it shares hardware with other VPS instances), but is otherwise a full-fledged (linux) server with the full power of the platform.

There are platform-as-a-service offers out there like heroku or fly.io and similar. These have the advantage that they handle much of the setup from databases to backups, but have the disadvantage that you are (a) tied to the platform with respect to costs and (b) need to learn platform-specific setup scripts.

Imho it's better to keep such dependencies at a minimum. While they promise to save time and costs, more often than not, they do neither, when free plans are suddenly discontinued and the setup interface changes often.

It's better to take a day or two to learn to set up things yourself. There's some initial learning cost, but for hobby projects you will be able to run a number of different projects on the same server that'll cost you just a couple of EUR per month.

In case one of your projects booms and requires a more serious setup to scale, you will be better informed about the options and can then migrate to a dedicated server.

A web app in the sense we are using it here can be for example a django or fastapi programm, a website with javascript and anything in between.

I'm not going into details of the setup here, but just go over a minimal configuration that allows you to run an app on the server. For a production server with serious traffic and business value you will need more configuration from ssl certificates and domain names over security and backup to various databases and maybe queuing systems, automated deploy and much more. For the sake of clarity I'll omit all this and concentrate only on essential steps:

1. [Server provider and image](#server-provide-and-image): Where to set up your VPS and which image to use
1. [Installation of software](#server-update-and-installation): Update the server and install basic software
1. [Web app environment](#web-app-runtime-environment): Create a runtime environment for the web app
1. [Upload web app](#upload-the-web-app): Get the code into place with the right permissions
1. [Install the venv](#install-the-webapp-poetry-venv): Install the direct runtime depencies (via poetry in this example)
1. [Running the app](#running-the-web-app): Running the webapp
1. [Monitor and (re)start](#monitor-and-auto-restart): Automate the (re)start of the runscript
1. [Optional webserver](#optional-nginx): Set up nginx for additional flexibility

## Server provide and image

I'm assuming you have a VPS up and running and that you have root access to it. Ideally take some time to set up ssh keys for easy, passwordless login. For the examples here, I assume that you are running ubuntu (server, i.e. without graphical user interface).

Your VPS provider (e.g. <https://www.netcup.de/>, <https://www.hetzner.com/>, <https://www.digitalocean.com/> to name a few) will typicall provide a starter image to get you going. Unless you have a compelling reason to pick something exotic stay with a popular image, simply since maintenance and (community) support will be better.

## Server update and installation

For our project we assume a [fastapi](https://fastapi.tiangolo.com/) app, that is configured with [python-poetry](https://python-poetry.org/) as environment manager.

Upgrade the server and install the packages we need:

{% highlight shell linenos %}

$ sudo apt update && apt upgrade && apt autoremove
[... output ...]
$ sudo apt install supervisor python-is-python3
[... output ...]
$ python --version
Python 3.10.12
$ sudo curl -sSL https://install.python-poetry.org | python3 -

{% endhighlight %}

The requirements on top of the server image are minimal. Depending on the type of web app that you run, the software requirements might be a bit different - for example you might require nginx, celery, postgresql etc. but my app just needs python and python-poetry.

The installation of `supervisor` is a convenience setup that isn't strictly necessary, but helpful in controlling the web app.

## Web app runtime environment

In good old unix tradition I'll run my web app as a dedicated user with limited system access. Since my server only has the job to run web apps and don't has any regular users, I'll also serve content from `/home/mywebapp/`. In a different setting I might choose something like `/var/mywebapp` or even `/var/webapps/mywebapp`.

{% highlight shell linenos %}

$ sudo groupadd --system webapp
$ sudo useradd --system --gid webapps --shell /bin/bash --home /home/mywebapp mywebapp
$ sudo mkdir /home/mywebapp
$ sudo chown mywebapp:webapps /home/mywebapp

{% endhighlight %}

{% highlight shell linenos %}

$ sudo su - mywebapp
mywebapp$ curl -sSL https://install.python-poetry.org | python3 -

Retrieving Poetry metadata

# Welcome to Poetry!

This will download and install the latest version of Poetry,
a dependency and package manager for Python.

It will add the `poetry` command to Poetry's bin directory, located at:

/home/mywebapp/.local/bin

You can uninstall at any time by executing this script with the --uninstall option,
and these changes will be reverted.

Installing Poetry (1.7.1): Done

Poetry (1.7.1) is installed now. Great!

To get started you need Poetry's bin directory (/home/mywebapp/.local/bin) in your `PATH`
environment variable.

Add `export PATH="/home/mywebapp/.local/bin:$PATH"` to your shell configuration file.

Alternatively, you can call Poetry explicitly with `/home/mywebapp/.local/bin/poetry`.

You can test that everything is set up by executing:

`poetry --version`

mywebapp$ vim .bashrc  # add the "export PATH" line from above
mywebapp$ source .bashrc
mywebapp$ poetry --version
Poetry (version 1.7.1)

{% endhighlight %}

## Upload the web app

At this point I have a webapp on my development computer, that I tested there locally. It doesn't really matter how you get the app onto your server, just make sure you upload it to the `/home/mywebapp` directory and set proper permissions.

In a more permanent setting I'd do this by either pulling the web app in via git manually or periodically or to set up a webhook that will be notified if a git-repo is updaten and will then take care of the deployment work (likely something like `git pull`, run tests, activate).

Here we keep it simply and just transfer our app with rsync. We are running the remote part of rsync by specifying `--rsync-path` so that we can use our login user for the ssh login, but change to our webapp user for saving the files (we run the remote rsync process as user `mywebapp`)(see [this stackexchange thread for details](https://unix.stackexchange.com/questions/240814/rsync-with-different-user)). From the development box:

{% highlight shell linenos %}

$ rsync --rsync-path 'sudo -u mywebapp rsync' -avP /home/cargocultprg/projects/mywebapp/ aera-incognita:/home/mywebapp/

{% endhighlight %}

You can test the command using the option `--dry-run` first, to make sure it lands right. By the end of this you should have the source code of your web app on the server in the directory `/home/mywebapp` with all files set to be owned by user `mywebapp`.

## Install the webapp (poetry) venv

As mentioned, I'm using python-poetry to manage the virtual environment for my webapp. Since I already installed poetry, I can now simply run `poetry install` to set up the virtual environment on the server.

First we want to make sure that the venv is created in our project root directory by setting the respective poetry setting true: `poetry config virtualenvs.in-project true`. To double check:

{% highlight shell linenos %}

$ poetry config --list
cache-dir = "/home/mywebapp/.cache/pypoetry"
experimental.system-git-client = false
installer.max-workers = null
installer.modern-installation = true
installer.no-binary = null
installer.parallel = true
virtualenvs.create = true
virtualenvs.in-project = true
virtualenvs.options.always-copy = false
virtualenvs.options.no-pip = false
virtualenvs.options.no-setuptools = false
virtualenvs.options.system-site-packages = false
virtualenvs.path = "{cache-dir}/virtualenvs"  # /home/mywebapp/.cache/pypoetry/virtualenvs
virtualenvs.prefer-active-python = false
virtualenvs.prompt = "{project_name}-py{python_version}"
warnings.export = true

{% endhighlight %}

The we can install the virtual environment, just like we used it in development:

{% highlight shell linenos %}

$ poetry install
Creating virtualenv mywebapp in /home/mywebapp/.venv
Installing dependencies from lock file

Package operations: 40 installs, 0 updates, 0 removals

  • Installing exceptiongroup (1.2.0)
  • Installing idna (3.6)
  • Installing sniffio (1.3.0)
  • Installing anyio (4.1.0)
  • Installing certifi (2023.11.17)
  ...

{% endhighlight %}

## Running the web app

With the virtual environment installed, I can activate it and run the web app. In my case it's a python fastAPI app that runs with uvicorn.

{% highlight shell linenos %}

$ poetry shell
(mywebapp-py3.10) mywebapp@blackbox:~/fastAPI$ uvicorn myapidir.myapi:app --reload 
INFO:     Will watch for changes in these directories: ['/home/mywebapp/fastAPI']
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [39036] using WatchFiles
INFO:     Started server process [39038]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
[Strg + C]
INFO:     Shutting down
INFO:     Waiting for application shutdown.
INFO:     Application shutdown complete.
INFO:     Finished server process [39038]
INFO:     Stopping reloader process [39036]

{% endhighlight %}

To automate this, we'll create a minimal shell script like this:

{% highlight bash linenos %}

#!/bin/bash
# /home/mywebapp/.local/bin/mywebapp_run

NAME="mywebapp"
APPDIR=/home/webapp/fastAPI/
USER=mywebapp

echo "starting $NAME as `whoami`"

# activate venv
cd $APPDIR

# run webapp
exec /home/mywebapp/.local/bin/poetry run uvicorn myapidir.myapi:app --reload

{% endhighlight %}

Copy it to `.local/bin/mywebapp_run`, set it executable `chmod u+x ~/.local/bin/mywebapp_run` and double check it, buy running it as your mywebapp user.

## Monitor and auto-restart

At this point I have a single executable that runs my app. Now I want to make sure that it is running on server restart. I set this up with `supervisor` which I installed at the start.

The following script tells supervisor what to do:

{% highlight bash linenos %}

#/etc/supervisor/conf.d/mywebapp.conf
[program:mywebapp]
command = /home/mywebapp/.local/bin/mywebapp_run
user = mywebapp
stdout_logfile = /home/mywebapp/fastAPI/logs/supervisor.log
redirect_stderr = true

{% endhighlight %}

Here's how to start supervisor:

{% highlight bash linenos %}

$ sudo systemctl enable supervisor
$ sudo systemctl restart supervisor
$ sudo supervisorctl status mywebapp

{% endhighlight %}

For every change of the script in `/etc/supervisor/conf.d/mywebapp.conf` supervisor must be restarted. Now supervisorctl can control the webapp: 

{% highlight bash linenos %}

$ sudo supervisorctl stop mywebapp
mywebapp: stopped
$ sudo supervisorctl start mywebapp
mywebapp: started
$ supervisorctl status mywebapp
mywebapp             RUNNING   pid 40368, uptime 0:01:14

{% endhighlight %}

## Optional: nginx

Since I used fastapi, the documentation is conveniently included via the openapi specification and can be accessed directly from the api. However in a different setting or additionally you may want to set up and nginx server to serve some static content like documentation.

Note, that it's also possible to use nginx as a front and wire it up to our api, so that everything is delivered through nginx. Such a setup would make it possible to serve - for example - different services from different domain names (but still from the single VPS).

However configuring this is beyond the aim of this post, so I leave it at that.