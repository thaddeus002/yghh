# Yannick's Git Hosting Howto

This document will describe one way to host Git repositories on a server.

There are many ways to do this, I present here my way, the one I think is simpler.

## Prerequisites

You'll need of course a server with root access. It will be named "example.com" in this document. In the following, replace "example.com" by the name or the IP address of your server.

We'll also used HTTP protocol for anonymous repositories clone. And so a web server must be install on "example.com". The configurations showned are for the apache server. This Howto doesn't present the configuration for others web servers. Please contribute if you want them to appear.

The git push, will be made over ssh protocol. The sshd deamon is probably already running on your server. If not, install it.

## Set the repositories

## Optional web interface with CGit

## Author

Yannick Garcia - 2018

## License

This document is release under a free license CC-By 4.0

