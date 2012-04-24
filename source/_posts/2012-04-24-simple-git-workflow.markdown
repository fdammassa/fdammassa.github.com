---
layout: post
title: "A simple git workflow"
date: 2012-04-24 11:49
comments: true
share: true
footer: true
categories: 
- git
- deploy
---

I have been working for some years with [svn](http://subversion.apache.org) and I was happy with it. But I have to admit that [git](http://git-scm.com) is very cool as well and I started using it a couple months ago. No regrets till now :)

In this post I don't want to list pros or cons of git versus svn or viceversa but I'd like to share a very simple worflow I've adopted to manage my live and dev environments and to keep them in sync through git.

<!-- more -->

## Setup ##
### Live Server ###

You need to create a bare repository (that we called newrepo.git) somewhere in your live server. I'm assuming that the <code>/var/git/repo</code> directory has been previously created.

``` bash
cd /var/git/repo
mkdir newrepo.git
cd newrepo.git
git init --bare
```

As second step, you should clone the newrepo.git to your webserver root directory, assuming this repo will contain the website files you are working on.

``` bash
cd /var/www
git clone /var/git/repo/newrepo.git newsite
```

### Development Server ###
On your development server, you have to clone the same repository newrepo.git. It is worth to note that the clone has been added the remote url automatically since you are cloning through ssh protocol.

``` bash
cd /var/www
git clone ssh://myuser@myliveserver.com/var/git/repo/newrepo.git newsite
```


## Deploy local changes ##

### Development Server ###
First of all, you should to prevent web access to the .git directory (differently from svn, it exitsts only at your clone root) by creating a <code>.htaccess</code> file at the same level of <code>.git</code> folder.

``` bash
cd /var/www/newsite
touch .htaccess
```

``` bash .htaccess
# deny access to the top-level git repository:
RewriteEngine On
RewriteRule \.git - [F,L]
```

Now add your first extra-complicated index file :)
``` bash
echo 'Hello World!' >> index.html
```

And sync the changes with your local repository.
``` bash
git add .
git commit -m ".git folder protected from web access"
```

Last step consists of pushing local changes to the remote repository (with 'remote repository' I mean the one on live-server at <code>/var/git/repo/newrepo.git</code> path).
``` bash
git push origin master
```


### Live Server ###
Finally, pull down changes from the newrepo.git repository to the clone in your web server root directory.
``` bash
cd /var/www/newsite
git pull origin master
```
