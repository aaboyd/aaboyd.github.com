---
layout: post
title: Windows Github and Bitbucket setup
date: 2013-08-04 17:57:47 -07:00
tags: [git, github, hg, bitbucket, windows, ssh, putty]
---

After I installed the Windows 8.1 update, and I kind of destroyed my machine and had to re-install Windows fresh.  This was pretty easy, but I lost all the configuration I had setup for developing with [Github](http://github.com) and [Bitbucket](http://bitbucket.org).

So, here is the quickest way I found to get setup and ready to contribute on [Github](http://github.com) and [Bitbucket](http://bitbucket.org).

## Installation of Tools

### SSH
To solve my ssh requirement, which I use for both git and mercurial, I simply installed [Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/), which I belive is pretty standard for ssh on windows.  I opted to go with the [Putty Installer](http://the.earth.li/~sgtatham/putty/latest/x86/putty-0.62-installer.exe).  It should install a few different tools that will be necessary in the future.

##### Chocolatey Alternative
{% highlight powershell %}
cinst putty
{% endhighlight %}

### Git
[Git for Windows]() has an [installer](http://code.google.com/p/msysgit/downloads/detail?name=Git-1.8.3-preview20130601.exe&can=2&q=full+installer+official+git) which makes the git installation pretty simple.  There are a few options that you can choose from and I recommend taking the time to actually read and learn about those options.  One that is sometimes required for projects is core.autocrlf to 'false', I would recommend checking this box during the installation.

##### Chocolatey Alternative
{% highlight powershell %}
cinst git
{% endhighlight %}

### Hg ( Mercurial )
[Mercurial](http://mercurial.selenic.com/downloads/), also, has an [installer](http://mercurial.selenic.com/release/windows/mercurial-2.7.0-x64.msi).  Mercurial doesn't have as many options as the Git install, but if you don't want "translations" or "documentation" you can leave those out of the install.

##### Chocolatey Alternative
{% highlight powershell %}
cinst hg
{% endhighlight %}

### GUI Client
If you like GUI based tools for source management, I would recommend [SourceTree](http://www.sourcetreeapp.com/), which I discovered when I started developing on a Macbook Pro.  It has since been ported to Windows and is getting regular updates.  It works well with both Git and Hg.  Also, allows you to connect your github and/or bitbucket account.  Of course there are alternatives, like tortoise GUI's ( [TortoiseGit](http://code.google.com/p/tortoisegit/) and [TortoiseHg](http://tortoisehg.bitbucket.org/) ).

## Configure Accounts

### SSH key generation
Open up puttygen, which was installed with the putty installer as well as the chocolatey install.  Click generate and then follow the instructions ( you have to move your mouse randomly ).  After the key is generated, you should save the private key and copy the public key info near the top of the puttygen dialog.  I chose to save them in a ```.ssh``` directory in my home directory ( this is consistent with Linux and Mac based systems ).

### Associate SSH key with account
Both git and bitbucket can use ssh for establishing identity.

#### Github
1. Login to your github account
2. Go to account settings from the top right corner.
3. On the left menu click ssh keys
4. Press 'Add Key'
5. Give the key an appropriate name
6. Paste the public key you copied earlier

#### Bitbucket
1. Login to your bitbucket account
2. From the top right dropdown select 'Manage Account'
3. From the left menu click 'SSH keys'
4. If you have linked your github and bitbucket accounts you can just press 'import keys from Github'
5. If you have not, you can click 'Add key'
6. Give the key an appropriate name
7. Paste the public key that you copied earlier

### Pageant SSH key on startup
If you do a lot of development and would rahter not have to worry about running pageant and loading the ssh key everytime you want to do something, than you can create a script in your "startup" folder.  To locate the startup folder the fastest method I have found is to press ```win + r``` and type ```shell:startup``` and press enter.  This will get you to the correct folder.  For me I made a simple "Start Pageant.bat" script, that contains the following :
{% highlight bat %}
@powershell -Command "start-process pageant -NoNewWindow -PassThru -ArgumentList "$env:HOMEDRIVE$env:HOMEPATH\.ssh\private.ppk"
{% endhighlight %}
**Note :** make sure to change the path to the key file so that the correct ppk file is used by pageant.

### SSH config for hg and git

#### Git
To use the correct agent for SSH with Git for Windows you need to set an environment variable.  Add an environment variable GIT_SSH=\Path\to\plink.exe.  Plink will use the ssh key that was loaded into pageant.

#### Mercurial
To use the plink with hg you must add it to the mercurial config file.  This is pretty simple, open up the mercurial.ini file that is located in your home directory.  Under the [ui] section, add
{% highlight bash %}ssh=plink -ssh{% endhighlight %}
Also, if you want command line capabilities with Mercurial you will need to add it to your PATH environment variables, this is not done during the installation.

## Start developing!
Now that you are ready to contribute, go make your mark on the internet community!

I just started to really try and get involved in some open source software.  It has been pretty rewarding and a great learning experience.  I would recommend it to any software engineer, inexperienced or expert.
