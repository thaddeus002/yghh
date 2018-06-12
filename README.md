# Yannick's Git Hosting Howto Project

This document will describe one way to host Git repositories on a server. IS STILL NOT COMPLETED.

There are many ways to do this, I present here my way, the one I think is simpler. This is a moving document that can evolute with time. You are free to follow this as a guide if you don't know how to do. Let me know if you think something is wrong, or make a pull request.

## Prerequisites

You'll need of course a server with root access. It will be named "example.com" in this document. In the following, replace "example.com" by the name or the IP address of your server.

We'll also used HTTP protocol for anonymous repositories clone. And so a web server must be install on "example.com". The configurations showned are for the apache server. This Howto doesn't present the configuration for others web servers. Please contribute if you want them to appear.

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

### configure web access to the repository (only need for anonymous clone) 

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

Create now an empty bare repository :

    # mkdir /var/www/htdocs/yghh.git
    # cd /var/www/htdocs/yghh.git
    # git init --bare

For the repository to be clonable by HTTP, you must add a hook for each update. This is not needed if you don't want your repository to de clonable by everyone.

    # mv hooks/post-update.sample hooks/post-update
    # ./hooks/post-update

### clone and push

From an other computer, you can do,

For read only access with HTTP protocol :

    $ git clone http://example.com/yanngit/yghh.git

or

    $ git clone yannick@example.com:/var/www/htdocs/yghh.git

### You did it

If you arrived here, your git hosting is operationnal. Now you can add repositories and users with write access to them. 

## Optional web interface with CGit

-- TO BE CONTINUED

## Author

Yannick Garcia - 2018

## License

This document is release under a free license CC-By 4.0

