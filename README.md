# Yannick's Git Hosting Howto Project

This document will describe one way to host Git repositories on a server. IS STILL NOT COMPLETED.

There are many ways to do this, I present here my way, the one I think is simpler. This is a moving document that can evolute with time. You are free to follow this as a guide if you don't know how to do. Let me know if you think something is wrong, or make a pull request.

## Prerequisites

You'll need of course a server with root access. It will be named "example.com" in this document. In the following, replace "example.com" by the name or the IP address of your server.

We'll also used HTTP protocol for anonymous repositories clone. And so a web server must be install on "example.com". The configurations showned are for the apache server. This Howto doesn't present the configuration for others web servers. Please contribute if you want them to appear.

The git push, will be made over ssh protocol. The sshd deamon is probably already running on your server. If not, install it.

Finally, git must be installed on this server

## Set the repositories

The repositories will be in "/var/www/htdocs". If the directory does not exits, create it :

    # mkdir -p /var/www/htdocs

The contents of this directory must be readable at least by the webserver, so the repositories will be clonable. I choosed let it be readable by everybody.

The repositories must also be writable by the people which are allowed to push their commits. As this will be made by ssh, these people will a accounts on the server. We'll name then "users".

The write rigths will be allowed to groups. First, 

    # addgroup git
    # chgrp git /var/www/htdocs
    # chmod 2775 /var/www/htdocs

## Optional web interface with CGit

## Author

Yannick Garcia - 2018

## License

This document is release under a free license CC-By 4.0

