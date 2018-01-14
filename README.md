# DigitalOcean - how to set up a Node.js application for production on ubuntu 16.04


## If you don't have one SSH key already set

[Generating a new ssh key](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/#generating-a-new-ssh-key)

## Set new Digital Ocean droplet WITH ssh login

[How to create your first DigitalOcean droplet](https://www.digitalocean.com/community/tutorials/how-to-create-your-first-digitalocean-droplet)

## Digital Ocean Networking settings 

- Add new domain e.g: example.com **without the www**
- Set **A** records for example.com


```
HOSTNAME		WILL DIRECT TO
@			droplet-ip
*			droplet-ip
www			droplet-ip
test			droplet-ip
```

More at  [How to set up a host name with digitalocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-host-name-with-digitalocean)

## SSH login:

`$ ssh root@droplet-ip`

## Set root UNIX password

`$ passwd`

## Allow firewall port 22 for ssh connections and Enable firewall 

```
$ ufw allow 22
$ ufw enable
```
I will ask: 
```
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```
## Add nginx repository

```
sudo add-apt-repository ppa:nginx/stable
sudo apt-get update
```

## Install nginx

`$ apt install nginx-full`

## Adjust the Firewall

`$ ufw status`

You will see: 

```
Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere                  
22 (v6)                    ALLOW       Anywhere (v6)             
```

List available options

`$ sudo ufw app list` 

You will see: 
```
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```
As you can see, there are three profiles available for Nginx:

**Nginx Full:** This profile opens both port 80 (normal, unencrypted web traffic) and port 443 (TLS/SSL encrypted traffic)
**Nginx HTTP:** This profile opens only port 80 (normal, unencrypted web traffic)
**Nginx HTTPS:** This profile opens only port 443 (TLS/SSL encrypted traffic)

Because the SSL is not set yet, we will add **Nginx Full** profile to accept not **Https** request.  

Add  Nginx Full and OpenSSH
```
$ ufw allow 'Nginx Full'
$ ufw allow OpenSSH
```

Check again `$ ufw status`

```
Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere                  
OpenSSH                    ALLOW       Anywhere                  
Nginx Full                 ALLOW       Anywhere                  
22 (v6)                    ALLOW       Anywhere (v6)             
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx Full (v6)            ALLOW       Anywhere (v6) 
```

More at:
[how to install nginx on ubuntu 16 04 - Adjust the firewall ](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04#step-2-adjust-the-firewall)
[How to setup a firewall with ufw on an ubuntu and debian cloud server](https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server)



## Nginx configured with SSL using Let's Encrypt certificates and Certbot

### Install
On Ubuntu systems, the Certbot team maintains a PPA. Once you add it to your list of repositories all you'll need to do is apt-get the following packages.

```
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-nginx 
```
### Setting up Nginx

Certbot can automatically configure SSL for Nginx, but it needs to be able to find the correct server block in your config. It does this by looking for a server_name directive that matches the domain you're requesting a certificate for.

If you're starting out with a fresh Nginx install, you can update the default config file. Open it with nano or your favorite text editor.

`$ sudo nano /etc/nginx/sites-available/default`

Find the existing server_name line and replace the underscore, _, with your domain name:

```
. . .
server_name example.com www.example.com test.example.com;
. . .
```
**You can save and close the file by hitting Ctrl-X, followed by Y, and then Enter to confirm.**

Then, verify the syntax of your configuration edits.

`$ sudo nginx -t`

If you get any errors, reopen the file and check for typos, then test it again.

Once your configuration's syntax is correct, reload Nginx to load the new configuration.

`$ sudo systemctl reload nginx`

More at: [How to secure nginx with let s encrypt on ubuntu 16-04 - Setting up nginx](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04#step-2-%E2%80%94-setting-up-nginx)

### Obtaining an SSL Certificate

**Hot issue:** https://github.com/certbot/certbot/issues/5405#issuecomment-356498627

> Unfortunately, Let's Encrypt has stopped offering the mechanism that Certbot's Apache and Nginx plugins use to prove you control a domain due to a security issue. See https://community.letsencrypt.org/t/2018-01-11-update-regarding-acme-tls-sni-and-shared-hosting-infrastructure/50188 for more info.

So to get SSL Certificate run this: 

```
$ sudo certbot --authenticator standalone --installer nginx --pre-hook "service nginx stop" --post-hook "service nginx start"
```
- Next it will ask for your email, "Enter email address (used for urgent renewal and security notices)"
- Then ask to agree to Terms of Service. **R: A**  
- Then ask to share your email with the Electronic Frontier
Foundation.  **R: Y**
- Then it will ask for which names would you like to activate HTTPS for? **R: 1,2,3** 
- Then it will ask for whether or not to redirect HTTP traffic to HTTPS, removing HTTP access. **R: 2**

Then you will see 
```
Congratulations! You have successfully enabled https://example.com, https://test.example.com, and https://www.example.com
```

At this point, you can see your site browser bar `https://www.example.com`, with **https** SSL.

## Install Node.js

### First, you need to install the NodeSource PPA in order to get access to its contents. Make sure you're in your home directory, and use curl to retrieve the installation script for the Node.js 6.x archives:

```
$ cd ~
$ curl -sL https://deb.nodesource.com/setup_6.x -o nodesource_setup.sh
```

### Run the script:

`$ sudo bash nodesource_setup.sh`

The PPA will be added to your configuration and your local package cache will be updated automatically. After running the setup script from nodesource, you can install the Node.js package in the same way that you did above:

`$ sudo apt-get install nodejs`

The nodejs package contains the nodejs binary as well as npm, so you don't need to install npm separately. However, in order for some npm packages to work (such as those that require compiling code from source), you will need to install the build-essential package:

`$ sudo apt-get install build-essential`

## Create Node.js Application

`cd /var/www/html/`

Create a file: `$ nano index.js` and save the next code: 

```
#!/usr/bin/env nodejs
var http = require('http');
http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello World\n');
}).listen(8080, 'localhost');
console.log('Server running at http://localhost:8080/');
```

### Install PM2 - which is a process manager for Node.js applications. PM2 provides an easy way to manage and daemonize applications (run them in the background as a service).

```
# Install latest PM2 version
$ sudo npm install pm2@latest -g
```

### Start Application

The first thing you will want to do is use the pm2 start command to run your application, hello.js, in the background:

`$ pm2 start index.js`

The startup subcommand generates and configures a startup script to launch PM2 and its managed processes on server boots:

`$ pm2 startup systemd`

The last line of the resulting output will include a command that you must run with superuser privileges:

```
[PM2] Init System found: systemd
[PM2] To setup the Startup Script, copy/paste the following command:
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u newuser --hp /home/newuser
```

More at https://github.com/Unitech/pm2

### Set Up Nginx as a Reverse Proxy Server

Open the file for editing:

`$ sudo nano /etc/nginx/sites-available/default`

Replace the contents of that block with the following configuration. If your application is set to listen on a different port, update the highlighted portion to the correct port number.

```
. . .
    location / {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**Before: **

```
location / {
	# First attempt to serve request as file, then
	# as directory, then fall back to displaying a 404.
	try_files $uri $uri/ =404;
}
```

**After**: 

```
location / {
	# First attempt to serve request as file, then
	# as directory, then fall back to displaying a 404.
	#try_files $uri $uri/ =404;
	proxy_pass http://localhost:8081;
	proxy_http_version 1.1;
	proxy_set_header Upgrade $http_upgrade;
	proxy_set_header Connection 'upgrade';
	proxy_set_header Host $host;
	proxy_cache_bypass $http_upgrade;
}
```

Make sure you didn't introduce any syntax errors by typing:

`$ sudo nginx -t`

Next, restart Nginx:

`$ sudo systemctl restart nginx`

More at  [how to set up a node js application for production on ubuntu 16 04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-16-04)

## Optimise 

Go to https://tools.pingdom.com, enter the domain: `https://www.example.com`, You will get **Performance grade of B 83**, To solve this, you need `Specify a Vary: Accept-Encoding header`. 

Open the file **nginx.conf** 

`$ nano /etc/nginx/nginx.conf`

**Uncomment**

``` 
        gzip_vary on;
        gzip_proxied any;
        gzip_comp_level 6;
        gzip_buffers 16 8k;
        gzip_http_version 1.1;
        gzip_types text/plain text/css application/json application/javascript text/xml application/xml$
```
**You can save and close the file by hitting Ctrl-X, followed by Y, and then Enter to confirm.**

Make sure you didn't introduce any syntax errors by typing:

`$ sudo nginx -t`

Next, restart Nginx:

`$ sudo systemctl restart nginx`

Test again: https://tools.pingdom.com, You should have Performance grade of **A 100**.

More at [Set vary accept encoding header nginx](https://stackoverflow.com/questions/6637678/set-vary-accept-encoding-header-nginx)

## Conclusion
Congratulations! You now have your Node.js application running behind an Nginx reverse proxy on an Ubuntu 16.04 server. This reverse proxy setup is flexible enough to provide your users access to other applications or static web content that you want to share. Good luck with your Node.js development!

# Appendix

## Get Familiar with Important Nginx Files and Directories

Now that you know how to manage the service itself, you should take a few minutes to familiarize yourself with a few important directories and files.

### Content
`/var/www/html:` The actual web content, which by default only consists of the default Nginx page you saw earlier, is served out of the `/var/www/html directory.` This can be changed by altering Nginx configuration files.

### Server Configuration

- `/etc/nginx:` The Nginx configuration directory. All of the Nginx configuration files reside here.

- `/etc/nginx/nginx.conf:` The main Nginx configuration file. This can be modified to make changes to the 

### Nginx global configuration.

- `/etc/nginx/sites-available/:` The directory where per-site "server blocks" can be stored. Nginx will not use the configuration files found in this directory unless they are linked to the sites-enabled directory (see below). Typically, all server block configuration is done in this directory, and then enabled by linking to the other directory.

- `/etc/nginx/sites-enabled/:` The directory where enabled per-site "server blocks" are stored. Typically, these are created by linking to configuration files found in the sites-available directory.

- `/etc/nginx/snippets:` This directory contains configuration fragments that can be included elsewhere in the Nginx configuration. Potentially repeatable configuration segments are good candidates for refactoring into snippets.

### Server Logs

- `/var/log/nginx/access.log:` Every request to your web server is recorded in this log file unless Nginx is configured to do otherwise.

- `/var/log/nginx/error.log:` Any Nginx errors will be recorded in this log.

More at [How to install nginx on ubuntu 16.04 - Get familiar with important nginx files and directories](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04#step-5-get-familiar-with-important-nginx-files-and-directories)

## Add a new user, and add to the sudo group:

```
$ adduser newuser
$ usermod -a -G sudo newuser
```
**(optional) Specifying Explicit User Privileges**

`$ visudo`

add this new line `newuser ALL=(ALL:ALL) ALL`, and save. 

```
root    ALL=(ALL:ALL) ALL
newuser ALL=(ALL:ALL) ALL
```
**You can save and close the file by hitting Ctrl-X, followed by Y, and then Enter to confirm.**

More at: 
[How to add and delete users on ubuntu 16 04](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-ubuntu-16-04)
[Initial server setup with ubuntu 16-04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04)
