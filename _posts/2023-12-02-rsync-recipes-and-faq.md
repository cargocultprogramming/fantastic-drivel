---
layout: post
title: rsync quick reference
description: Frequent rsync commands and common confusion
tags: cli linux
categories: 
featured: true
---
`rsync` is the go-to solution to quickly synchronize files between systems. It's pretty versatile, but with that power also comes complexity in it's usage. So let's go over some common ones and clear up some confusion on the way.

## common rsync options

TODO: important options


`--dry-run, -n`
: Dry run only simulates what would happen and is a good option to use if you are not sure or if an error would have bad consequences (such as when using `--delete`). Pipe the command through `grep del` to make sure to catch any delete actions.

`--archive, -a`
: Archive mode, is a pretty common mode and a short cut of a bunch of other modes (`-rlptgoD`). It basically means preserve all file attributes (almost all) and do it recursively. It's the basic go-to option for syncing directories.

`--delete`
: Delete files the exist at destination but not at origin. Be careful with this one - if you mix up trailing slashes for example this will potentially delete a large number of files.

`--update, -u`
: Ignores all files that exist on the destination and have a newer mtime than the file on the origin. In other words, this only updates older files on the destination. (If the mtime is equal, but size is different the destination will also be updated.)

`--ignore-existing`
: Ignores all existing files, but will update the destination with any new files that exist at the origin. This does not ignore directories.

`--filter 'this'`
: Filters out expressions that will not be synchronized from origin to destination. For example directories or files - also regular expressions are possible. For multiple strings just use the filter option repeatedly.

+ `-avz`

## rsync easy ssh

`rsync` works over a number of protocols, most commonly ssh. With a little configuration this becomes very convenient. If you are using ssh public/private key pairs to log into your different boxes, you can easily set up shortcuts for these in `.ssh/config`. Here is an example:

{% highlight config linenos %}
#.ssh/config
Host blackbox
        Hostname 68.89.147.230
        User johnathan.surname

Host pandora
        Hostname 86.98.174.305
        User john

Host pandora-local
        Hostname 192.168.0.7
        User john

Host pandora-local-private
        Hostname 192.168.0.7
        User js

{% endhighlight %}

These lines basically set up shortcuts for the host/username-combination, so you don't have to remember them yourself - just set and forget. With the keys, the login becomes effectively passwordless if the keys are available via ssh-agent for example (I'll cover that in another post).

The nifty thing in combination with `rsync` is, that once set up, this will enable path completion for your rsync command. So entering something like `rsync blackbox:~/` and hitting tab will automatically expand to `rsync blackbox:/home/jonathan.surname/`, add the first couple of letters of the directory you want to sync from, hit tab again and ssh will look up possible path completions in the the background. You can even hit tab twice after the slash and ssh will present you with a list of paths that are available, all without ever leaving the rsync line.

## rsync and trailing slashes

A common confusion with `sync` is weather to use trailing slashes or not on the *source* directory (trailing slash presence doesnt matter for the destination directory).

Here is a little mnemonic about this: if there is only the name of the directory itself *without* the slash, then the directory is the main thing, you name **the_directory**. But if you add a trailing slash, the directory that you named is not the main thing anymore - the slash adds a directional meaning **from within** the directory*.

`some_dir` (w/o trailing `/`): **the_directory**

`some_dir/` (with trailing `/`): from **within** the directory

So simply put: The slash adds direction.
