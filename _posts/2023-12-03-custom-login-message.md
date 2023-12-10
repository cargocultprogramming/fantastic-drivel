---
layout: post
title: Custom shell login message
description: Display a custom shell login message for ssh
tags: linux cli
categories: 
featured: true
---
If you log into different servers with ssh frequently it may make sense to display a custom welcome message for the server.

This is extremely easy to set up and helps to stay aware of your (shell) surroundings, so to speak.

To implement this, I like to display the ascii art of the server nickname as welcome message. I'm usually using the same nickname that I also use in my ssh configuration - for the sake of this example, we'll assume the name is `blackbox`.

First head over to <https://patorjk.com/software/taag> or <https://www.asciiart.eu/text-to-ascii-art>, enter the nickname, pick a style you like and copy the result. It might look something like this:

```ascii
 _     _            _    _               
| |   | |          | |  | |              
| |__ | | __ _  ___| | _| |__   _____  __
| '_ \| |/ _` |/ __| |/ / '_ \ / _ \ \/ /
| |_) | | (_| | (__|   <| |_) | (_) >  < 
|_.__/|_|\__,_|\___|_|\_\_.__/ \___/_/\_\

```

Now all you need to do is to stick this into `/etc/motd` (create it, if it doesnt exist) and you are good to go. The ascii label will be included at the end of the usual login message so your login will look something like this:

```ascii
$ ssh blackbox

Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-89-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Dec  3 05:42:01 PM UTC 2023

  System load:  0.0               Processes:               129
  Usage of /:   3.0% of 74.79GB   Users logged in:         1
  Memory usage: 3%                IPv4 address for enp7s0: 10.0.0.2
  Swap usage:   0%                IPv4 address for eth0:   86.35.12.232

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

1 update can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

 _     _            _    _               
| |   | |          | |  | |              
| |__ | | __ _  ___| | _| |__   _____  __
| '_ \| |/ _` |/ __| |/ / '_ \ / _ \ \/ /
| |_) | | (_| | (__|   <| |_) | (_) >  < 
|_.__/|_|\__,_|\___|_|\_\_.__/ \___/_/\_\

Last login: Sun Dec  3 17:04:45 2023 from 106.23.107.216

cargocultprg@blackbox:~#
```


For a more fine-grained configuration, take a look at the files `/etc/issue` and `/etc/issue.net` as well as the `banner` setting in `/etc/ssh/sshd_config`.

`motd` is displayed after login at the end of all other login messages, `issue` is displayed before login for local users and `issue.net` is the equivalent for users that connect via network.
