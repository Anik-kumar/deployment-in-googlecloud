# install gcloud in ubuntu
https://cloud.google.com/sdk/docs/quickstart-debian-ubuntu

# deployment steps
1. create vm instance in google cloud
2. install dependencies in vm instance
	1. nodejs
3. install reverse proxy, nginx
4. expose port 8080 for node server and mongodb port 27017
4. upload back-end and front-end code in vm instance
5. install mongodb
6. install a process manager, pm2
7. run front-end and back-end in process manager
8. configure load balancer
9. expose public ip
10. dns mapping


# install nodejs 12
```
$ sudo apt update //update repository
$ sudo apt -y upgrade
$ sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
$ curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
$ sudo apt -y install nodejs
```

# install nginx 
install cmd
```shell script
$ sudo apt install nginx
```
The version installed in my computer is `nginx/1.14.0 (Ubuntu)`\
Once the installation is completed, Nginx service will start automatically. You can check the status of the service:
```
$ sudo systemctl status nginx 
```
Start nginx server
```
$ sudo systemctl start nginx
```
Stop nginx server
```
$ sudo systemctl stop nginx
```
Restart nginx server
```
$ sudo systemctl restart nginx
```
Reload nginx when configuration changes
```
$ sudo systemctl reload nginx
```
nginx configuration file location
/etc/nginx/nginx.conf


https://linuxize.com/post/how-to-install-nginx-on-ubuntu-18-04/

# open port 8080
Default port 80
```
server {
	listen		80;
	server_name	localhost;
	
	root /var/www/html;
	location / {
            	#index  index.html index.htm;
		try_files $uri $uri/ /index.html;
        }
}
```
Create a new virtual host config file in /etc/nginx/sites-available/server2
```
server {
	listen		8080;
	server_name	localhost;
	
	root /var/www/html;
	location / {
            	#index  index.html index.htm;
		try_files $uri $uri/ /index.html;
        }
}
```
Add new server config in nginx.conf file
```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {
	....
	....
	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-available/server2;
	include /etc/nginx/sites-enabled/*;
}
```
# create a new firewall rules for port 8080
![google cloud firewall](/images/firewall1.png)

![google cloud firewall](/images/firewall2.png)

# upload files in google cloud using gcloud
```gcloud compute scp local-file-path instance-name:~```

Replace the following:
* `local-file-path`: The path to the file on your workstation.
* `instance-name`: The name of your instance.

Example
```
gcloud compute scp /home/user/workspace/upload/dist.zip server1:~
```
This zip file contains angular application.

Now we have to extract zip and place content in a directory where nginx default servers `root` is pointed. In this example , the root path is `/home/anik/uiapp`

Here is directory structure of `nginx 1.14` in `Ubuntu 18.04`
```
|-etc/nginx
|		|- conf.d
|		|- fastcgi.conf
|		|- fastcgi_params
|		|- koi-utf
|		|- koi-win
|		|- mime.types
|		|- modules-available/
|		|- modules-enabled/
|		|- nginx.conf
|		|- proxy_params
|		|- scgi_params
|		|- sites-available/
|				|- default
|				|- server2
|				|- server3
|		|- sites-enable/
|		|- snippets/
|		|- uwsgi_params
|		|- win-utf
```

`nginx.conf` file
```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-available/server2;
	include /etc/nginx/sites-enabled/*;
}
```
`etc/nginx/sites-enabled/` points to sites `etc/nginx/sites-available/`\
Virtual servers configured inside `sites-available` directory.

etc/nginx/sites-available/default 
```
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
#
# In most cases, administrators will remove this file from sites-enabled/ and
# leave it as reference inside of sites-available where it will continue to be
# updated by the nginx packaging team.
#
# This file will automatically load configuration files provided by other
# applications, such as Drupal or Wordpress. These applications will be made
# available underneath a path with that package name, such as /drupal8.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
	listen 80 default_server;
	listen [::]:80 default_server;

	# SSL configuration
	#
	# listen 443 ssl default_server;
	# listen [::]:443 ssl default_server;
	#
	# Note: You should disable gzip for SSL traffic.
	# See: https://bugs.debian.org/773332
	#
	# Read up on ssl_ciphers to ensure a secure configuration.
	# See: https://bugs.debian.org/765782
	#
	# Self signed certs generated by the ssl-cert package
	# Don't use them in a production server!
	#
	# include snippets/snakeoil.conf;

	#root /var/www/html;
	root /home/anik/uiapp;

	# Add index.php to the list if you are using PHP
	index index.html index.htm index.nginx-debian.html;

	server_name _;

	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}

	# pass PHP scripts to FastCGI server
	#
	#location ~ \.php$ {
	#	include snippets/fastcgi-php.conf;
	#
	#	# With php-fpm (or other unix sockets):
	#	fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
	#	# With php-cgi (or other tcp sockets):
	#	fastcgi_pass 127.0.0.1:9000;
	#}

	# deny access to .htaccess files, if Apache's document root
	# concurs with nginx's one
	#
	#location ~ /\.ht {
	#	deny all;
	#}
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
```
In my example, I have modified the root path, `/var/www/html`,  in default server which is running on port 80
```	
#root ;
root /home/anik/uiapp; 
```

Here is the directory structure of the angular app:
```
|- /home/anik/uiapp
|		|- index.html
|		|- main-es2015.4b2252f619f4b79f2cf3.js
|		|- main-es5.4b2252f619f4b79f2cf3.js
|		|- polyfills-es2015.564b7db4155506889ff6.js
|		|- polyfills-es5.ffd64b1dc88d29d4347c.js
|		|- runtime-es2015.0e9886773ddebcd849a9.js
|		|- runtime-es5.0e9886773ddebcd849a9.js
|		|- styles.5a46c7ae65db67990354.css
.....
.....
```

