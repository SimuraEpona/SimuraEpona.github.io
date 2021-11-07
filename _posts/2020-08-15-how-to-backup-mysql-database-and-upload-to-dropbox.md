---
id: 38
title: how to backup mysql database and upload to dropbox
date: 2020-08-15T11:37:39+09:00
author: epona
layout: post
guid: https://wp.epona.me/?p=38
permalink: /2020/08/15/how-to-backup-mysql-database-and-upload-to-dropbox/
categories:
  - PROGRAMMING
tags:
  - backup
  - linux
---
in this post, I will show you how to backup MySQL database in docker and upload it to [Dropbox](https://www.dropbox.com).

## install dropbox on Linux

we can install Dropbox through the command below:

<pre><code class="bash"># 32bit
$ cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86" | tar xzf -

# 64 bit

$ cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86_64" | tar xzf -
</code></pre>

then you can type the command `~/.dropbox-dist/dropboxd` to install dropbox.

but when I type `dropboxd` command, the error showed.

<pre><code class="log">Traceback (most recent call last):
  File "dropbox/client/main.pyc", line 254, in &lt;module&gt;
  File "dropbox/foundation/navigation_service/factory.pyc", line 22, in &lt;module&gt;
  File "dropbox/foundation/navigation_service/navigation_service_impl.pyc", line 57, in &lt;module&gt;
  File "dropbox/foundation/html_views/electron/manager_factory.pyc", line 14, in &lt;module&gt;
  File "dropbox/foundation/html_views/local/common/manager.pyc", line 33, in &lt;module&gt;
  File "dropbox/client/features/model_registry.pyc", line 13, in &lt;module&gt;
  File "dropbox/client/features/generated_models.pyc", line 292, in &lt;module&gt;
  File "dropbox/client/features/previews/view_anchor.pyc", line 106, in &lt;module&gt;
  File "&lt;_bootstrap_overrides&gt;", line 153, in load_module
ImportError: libglapi.so.0: cannot open shared object file: No such file or directory
!! dropbox: fatal python exception:
['Traceback (most recent call last):\n', '  File "dropbox/client/main.pyc", line 254, in &lt;module&gt;\n', '  File "dropbox/foundation/navigation_service/factory.pyc", line 22, in &lt;module&gt;\n', '  File "dropbox/foundation/navigation_service/navigation_service_impl.pyc", line 57, in &lt;module&gt;\n', '  File "dropbox/foundation/html_views/electron/manager_factory.pyc", line 14, in &lt;module&gt;\n', '  File "dropbox/foundation/html_views/local/common/manager.pyc", line 33, in &lt;module&gt;\n', '  File "dropbox/client/features/model_registry.pyc", line 13, in &lt;module&gt;\n', '  File "dropbox/client/features/generated_models.pyc", line 292, in &lt;module&gt;\n', '  File "dropbox/client/features/previews/view_anchor.pyc", line 106, in &lt;module&gt;\n', '  File "&lt;_bootstrap_overrides&gt;", line 153, in load_module\n', 'ImportError: libglapi.so.0: cannot open shared object file: No such file or directory\n'] (error 3)
Aborted (core dumped)
</code></pre>

the reason is that we are missing some extensions, probably are these below:

    sudo apt-get install libc6
    sudo apt-get install libglapi-mesa
    sudo apt-get install libxdamage1
    sudo apt-get install libxfixes3
    sudo apt-get install libxcb-glx0
    sudo apt-get install libxcb-dri2-0
    sudo apt-get install libxcb-dri3-0
    sudo apt-get install libxcb-present0
    sudo apt-get install libxcb-sync1
    sudo apt-get install libxshmfence1
    sudo apt-get install libxxf86vm1
    

after that, we should also make sure of the following:

    sudo chown "$USER" "$HOME"
    sudo chown -R "$USER" ~/Dropbox ~/.dropbox
    sudo chattr -R -i ~/Dropbox
    sudo chmod -R u+rw ~/Dropbox ~/.dropbox
    

this time we can successfully run the command `~/.dropbox-dist/dropboxd`. when we first run this command the output will show us an HTML link to request a link to our Dropbox account. after that, we have successfully installed Dropbox on Linux.

Last, we can download the official python [dropbox.py](https://www.dropbox.com/download?dl=packages/dropbox.py) to control dropbox. available commands are

    $ python3 dropbox.py
    Dropbox command-line interface
    
    commands:
    
    Note: use dropbox help <command> to view usage for a specific command.
    
     autostart    automatically start Dropbox at login
     exclude      ignores/excludes a directory from syncing
     filestatus   get current sync status of one or more files
     help         provide help
     lansync      enables or disables LAN sync
     ls           list directory contents with current sync status
     proxy        set proxy settings for Dropbox
     puburl       get public url of a file in your Dropbox's public folder
     running      return whether Dropbox is running
     sharelink    get a shared link for a file in your Dropbox
     start        start dropboxd
     status       get current status of the dropboxd
     stop         stop dropboxd
     throttle     set bandwidth limits for Dropbox
     update       download latest version of Dropbox
     version      print version information for Dropbox
    

## backup MySQL database

backup MySQL data from docker is very simple.

<pre><code class="bash"># Backup
docker exec CONTAINER /usr/bin/mysqldump -u root --password=root DATABASE &gt; ~/Dropbox/backup.sql

# Restore
cat backup.sql | docker exec -i CONTAINER /usr/bin/mysql -u root --password=root DATABASE
</code></pre>

## conclusion

firstly, we installed Dropbox to our Linux system. then, we use the docker command to dump our backup file to&nbsp;the `~/Dropbox` directory so that Dropbox can automatically upload to the Cloud server.

## references

  * [mysql-workder.sh](https://gist.github.com/spalladino/6d981f7b33f6e0afe6bb)
  * [install Dropbox on Linux](https://www.dropbox.com/install-linux)
  * [Headless install on Debian: libglapi.so.0: cannot open shared object file](https://www.dropboxforum.com/t5/Dropbox-installs-integrations/Headless-install-on-Debian-libglapi-so-0-cannot-open-shared/td-p/396457)