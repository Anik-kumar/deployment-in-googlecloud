# install gcloud in ubuntu
https://cloud.google.com/sdk/docs/quickstart-debian-ubuntu

# Deployment steps
1. create firewall rules vm instance in google cloud
2. create vm instance in google cloud
3. install dependencies in vm instance
	1. nodejs
4. install reverse proxy, nginx
5. expose port 8080 for node server and mongodb port 27017
6. upload back-end and front-end code in vm instance
7. install mongodb
8. install a process manager, pm2
9. run front-end and back-end in process manager
10. configure load balancer
11. expose public ip
12. dns mapping



# Create a new firewall rules for port 8080 and 27017
creating firewall rule for both port 8080 and 27017 using `Network tags: ` `mongodb-server`.

![google cloud firewall](/images/create-firewall1.png)
![google cloud firewall](/images/create-firewall2.png)
![google cloud firewall](/images/create-firewall3.png)

# Create vm instance
![google cloud vm instance](/images/create-vm-1.png)
![google cloud vm instance](/images/create-vm-2.png)
![google cloud vm instance](/images/create-vm-3.png)
![google cloud vm instance](/images/create-vm-4.png)
![google cloud vm instance](/images/create-vm-5.png)
![google cloud vm instance](/images/create-vm-7.png)
![google cloud vm instance](/images/create-vm-8.png)

# Connect to vm-instance via local cmd prompt
![google cloud vm instance](/images/connect-vm-1.png)
![google cloud vm instance](/images/connect-vm-3.png)
![google cloud vm instance](/images/connect-vm-4.png)
![google cloud vm instance](/images/connect-vm-5.png)

# Install nodejs 12
```
$ sudo apt update //update repository
$ sudo apt -y upgrade
$ sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
$ curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
$ sudo apt -y install nodejs
```

# Install nginx 
install command
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


# Open port 8080 in nginx
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

# Creating a new VM instance for mongodb
![google cloud vm instance](/images/create-vm-mongo1.png)
![google cloud vm instance](/images/create-vm-mongo2.png)
Add `mongodb-server` tag to the instance.


# Install mongodb
Connected to database-server from my local terminal.
![google cloud vm instance](/images/connect-database1.png)

\
Install commands:
```
$ sudo apt-get install gnupg
$ wget -qO - https://www.mongodb.org/static/pgp/server-4.2.asc | sudo apt-key add -
$ echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.2.list
$ sudo apt-get update
$ sudo apt-get install -y mongodb-org
```
`To install mongodb specific version.`
```
$ sudo apt-get install -y mongodb-org=4.2.6 mongodb-org-server=4.2.6 mongodb-org-shell=4.2.6 mongodb-org-mongos=4.2.6 mongodb-org-tools=4.2.6
```

Mongodb installed via package manager, data directory `/var/lib/mongodb` and log directory `/var/lib/mongodb` are created automatically.\
Mongodb config file is in `/etc/mongod.conf`.

# Configure mongod.conf
After connecting to mongodb vm-instance:
```
$ sudo nano /etc/mongod.conf
```
Change `bindIp: 127.0.0.1` to `bindIpAll: true`.\
Now restart mongod service & check status:
```
$ sudo systemctl restart mongod
$ sudo systemctl status mongod
```
If status is like this:
![google cloud vm instance](/images/mongoError.png)

Then run this cmd:
```
$ sudo /usr/bin/mongod  --config /etc/mongod.conf --fork
```
* This command will create a child process of `mongod` service each time you run.

# Create a admin user in database
* ### Start MongoDB without access control
```
$ mongod --port 27017 --dbpath /var/lib/mongodb
```
* ### Connect to the instance
```
$ mongo --port 27017
```
* ### Create the user administrator
```
$ use admin
$ db.createUser(
  {
    user: "myUserAdmin",
    pwd: "myUserPass", // or passwordPrompt()
    roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
  }
)
```
* ### Re-start the MongoDB instance with access control\
Exit mongo shell and run this:
```
$ sudo systemctl restart mongod
```
* ### Again change mongod configuration to enable access control
```
$ sudo nano /etc/mongod.conf
```
* ### Add this to mongod.conf file
```
security:
  authorization: enabled
```

* ### Again exit shell and restart mongod
```
$ sudo systemctl restart mongod
```
* ### Connect and authenticate as the user administrator
```
$ mongo --port 27017  --authenticationDatabase "admin" -u "myUserAdmin" -p "myUserPass"
```
> MongoUri format: `mongodb://<username>:<password>@<host>:<port>/<database>?authSource=<database>`

* ### Create new user for database
```
$ use test
$ db.createUser(
  {
    user: "myTester",
    pwd:  "myPass",   // or passwordPrompt()
    roles: [ { role: "readWrite", db: "test" },
             { role: "read", db: "reporting" } ]
  }
)
```
After creating the additional users, disconnect the mongo shell
* ### Connect to the instance and authenticate as myTester
```
$ mongo --port 27017 -u "myTester" --authenticationDatabase "test" -p "myPass"
```
* ### Insert a document as myTester
```
$ db.foo.insert( { x: 1, y: 1 } )
```

