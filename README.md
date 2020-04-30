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

#open port 8080
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
#create a new firewall rules for port 8080
![google cloud firewall](/images/firewall1.png)

![google cloud firewall](/images/firewall2.png)

