# Yannick's Git Hosting Howto Project

This document will describe one way to host Git repositories on a Linux server. IS STILL NOT COMPLETED.

There are many ways to do this, I present here my way, the one I think is simpler. This is a moving document that can evolute with time. You are free to follow this as a guide if you don't know how to do. Let me know if you think something is wrong, or make a pull request.

Must of the actions to do are generic, but, following your distribution, some paths or filename may change.

## Prerequisites

You'll need of course a server with root access. It will be named "example.com" in this document. In the following, replace "example.com" by the name or the IP address of your server.

We'll also used HTTP protocol for anonymous repositories clone. And so a web server must be install on "example.com". The configurations showned are for the apache server. This Howto doesn't present the configuration for others webservers. Please contribute if you want them to appear.

The git push, will be made over ssh protocol. The sshd deamon is probably already running on your server. If not, install it.

Finally, git must be installed on this server

## Set the repositories

### set the context

The repositories will be in "/var/www/htdocs". If the directory does not exits, create it :

    # mkdir -p /var/www/htdocs

The contents of this directory must be readable at least by the webserver, so the repositories will be clonable. I choosed let it be readable by everybody.

The repositories must also be writable by the people which are allowed to push their commits. As this will be made by ssh, these people will a accounts on the server. We'll name then "users".

The write rigths will be allowed to groups. First, we create a group "git" which will own by default all the repositories. For this, we put the "setguid" bit on their parent directory :

    # addgroup git
    # chgrp git /var/www/htdocs
    # chmod 2775 /var/www/htdocs

### configure web access to the repositories (only need for anonymous clone)

Add to the file "/etc/httpd/conf/httpd.conf" (may be "/etc/apache2/apache.conf" on a raspberry pi) :

    Alias /yanngit/ "/var/www/htdocs/"
    <Directory /var/www/htdocs>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

Then restart apache deamon :

    # service httpd restart

or

    # service apache2 restart

following apache's version.

### create a repository

To create the repository "yghh" owned by "yannick", first create the user "yannick" with group "git".

    # adduser yannick git

To prevent this user to obtain a shell (through ssh) on the server, configure git-shell as his login shell.

If git-shell is not in /etc/shells, first add it :

    # which git-shell >> /etc/shells

Then change yannick's shell :

    # chsh git -s `which git-shell`

Optionnaly put his public key in ~yannick/.ssh to not be asked for password. Or put an password with command "passwd yannick".

Create now an empty bare repository, and add write rights to group :

    # mkdir /var/www/htdocs/yghh.git
    # cd /var/www/htdocs/yghh.git
    # git init --bare
    # chmod -R 775 branches objects refs
    # chmod 775 info
    # chmod 664 HEAD info/*

For the repository to be clonable by HTTP, you must add a hook for each update. This is not needed if you don't want your repository to de clonable by everyone.

    # mv hooks/post-update.sample hooks/post-update
    # ./hooks/post-update
    # chmod 664 info/* objects/info/*

### clone and push

From an other computer, you can do,

For read only access with HTTP protocol :

    $ git clone http://example.com/yanngit/yghh.git

or

    $ git clone yannick@example.com:/var/www/htdocs/yghh.git

### You did it

If you arrived here, your git hosting is operationnal. Now you can add repositories and users with write access to them.

## Optional web interface with cgit

Now you have a server for hosting yours git repositories. And there is a webserver running on it.

So, you may like to browse your repositories and explore the commits in a web interface.

A choise to do this may be cgit. cgit is a cgi program. This mean it is execute when the webserver receive a HTTP GET request for the file "cgit.cgi".

For this to work the cgi module must be loaded by apache deamon, and the directory where is the cgi program must be declared as cgi directory. So the web server will execute the program instead of send it to your browser. What will be send is the output generated by the program.

### Apache server configuration for a CGI program

#### Loading cgi module

In your "/etc/httpd/conf/httpd.conf" or "/etc/apache2/apache2.conf" file uncomment the line beginning by "LoadModule cgid_module"

On a raspberry pi, the equivalent action is made by creating two symlinks :

    $ cd /etc/apache2/mods-enabled/
    $ ln -s ../mods-available/cgid.load .
    $ ln -s ../mods-available/cgid.conf .

#### Adding the cgi directory

add the following in "/etc/httpd/conf/httpd.conf" or "/etc/apache2/apache2.conf" file :

    ScriptAlias /cgit/ "/var/www/htdocs/cgit/"
    <Directory /var/www/htdocs/cgit>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

and create the directory :

    # mkdir /var/www/htdocs/cgit

#### Restarting service

    # service httpd restart

or

    $ sudo service apache2 restart

### Compiling and installing cgit

It's probably possible with your distribution to install cgit with the packet manager. But for better understand how it works, we'll build and install it ourself. If you don't want to do this, or don't have gcc installed on your server, skip this section and simply do

    # apt get install cgit

or

    # yum install cgit

or whatever is suitable for your distribution.

Else you can build cgit :

    $ git clone --recursive https://git.zx2c4.com/cgit
    $ cd cgit
    $ make NO_LUA=1

After that, install the program on the webserver :

    # cp cgit /var/www/htdocs/cgit/cgit.cgi

At this point, the url "http://example.com/cgit/cgit.cgi" must be accessible and present some text.

To make this page nicer, we need to add a css, a logo, and a favicon. Considering the root webserver directory is "/var/www/html", we can put theses files in it :

    # cp cgit*.css /var/www/html/
    # cp cgit.png /var/www/html/
    # cp favicon.ico /var/www/html/

By default, these file are looked for in the root directory on the web server. This can be override in by cgit configuration. See cgitrc manpage for detail.

To avoid error logs for apache, create the directory /var/cache/cgit and make it writable for apache

    # mkdir -p /var/cache/cgit
    # chgrp www-data /var/cache/cgit
    # chmod 775 /var/cache/cgit

### Configure your repository

You must declare yours repositories for cgit to be able to show them. For our example, we'll create a file "/etc/cgitrc" with the following content :

    clone-url=http://example.com/yanngit/$CGIT_REPO_URL.git
    repo.url=yghh
    repo.path=/var/www/htdocs/yghh.git
    repo.desc=Yannick's Git Hosting Howto
    repo.owner=Yannick Garcia  

You may have as much "repo.*" lines as repositories you are hosting.

If you select the project "yghh" in the list of projects now appearing on "http://example.com/cgit/cgit.cgi", you may see something looking like this :

![screenshot](yghh_view.png)

### Used cgit filters


-- TO BE CONTINUED

## References

For more informations on git hosting or on cgit, read the following references

  1. [Progit v2](https://git-scm.com/book/en/v2) by Scott Chacon, chapter 4 : Git on the server
  2. Cgit README, and manpage on [cgit website](https://git.zx2c4.com/cgit/)

## Author

Yannick Garcia - 2018

## License

![Creative Commons License](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)

This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License.

To view a copy of this license, visit [http://creativecommons.org/licenses/by-sa/4.0/](http://creativecommons.org/licenses/by-sa/4.0/) or send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.
