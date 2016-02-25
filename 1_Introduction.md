## Introduction 
What is docker and how it differs from Virtual machines you might ask yourself. 

Docker is a container technology, it's much more light weight than VMs and it allows us to package the software and all its dependencies into a container and we can move those containers from host to host without having to worry about missing a dependency or having a different dependency installed on the target system. 

To illustrate this point I have borrowed the following diagrams from Docker website. 

![vm](/images/what-is-vm-diagram.png)

The main difference here is we are not replicating guest OS between each and every container, this makes containers more 
portable and lightweight. 

Each container is meant to be running a single service, and it will exit automatically once that process exits as well.

![docker](images/what-is-docke-diagram.png)

### Docker Hub  
Docker hub is a public repository for all the container images, you can compose your new images based on existing images and this is what makes docker truly versatile.  

Images are normally composed of layers each layer is denoted as a single operation in the Dockerfile, you can think of it as a single commit after each operation.  
So lets say if you installed some software on top of clean ubuntu image and you had to push to Docker Hub you will only be transferring the diff and the reference to the original ubuntu base image.  

### Description of the project  

We will setup a Ghost blogging platform backed by MariaDb. 
Since its not a good idea to have node exposed to the internet we will install nginx reverse proxy in front of it. 

The whole setup shall consist of 3 Docker containers  
1. Ghost Container  
2. MariaDb Container  
3. Nginx Proxy  Container  
4. Data Container (optional)  

### Docker installation  

The simplest way to get docker on to your machine is to use the installation script provided by Docker 
Make sure you have curl installed, then run this as root  
```
$curl -fsSL https://get.docker.com/ | sh
```
Follow the onscreen instructions and at the end change add docker user to your user group to avoid always having
 to type sudo before the docker command, also its not recommended to run docker as a root user. 

```
usermod -aG docker max  
```
Here we have added max (my user) to the docker group and now we can execute all docker commands without having elevated privileges. 


### Setting up Ghost Docker container.   
We will first search for an official ghost docker image on docker hub.  
You can pull any image using docker pull {imageName}  as shown below  
```
docker  pull ghost
latest: Pulling from ghost

381453eac6fd: Pulling fs layer 
8a530b2f7425: Download complete 
193618932e96: Download complete 
aedc4b6a61ca: Download complete 
...
... 
Digest: sha256:a578f5fef2d77bbdff73b4413dbbbe66055ffcff0cb2d6c774270fc6e3918a58
Status: Downloaded newer image for ghost:latest

```
 
To see the ghost blogging platform in action with zero setup just issue 

```
docker run --name blog -p 2368:2368 -v /home/max/dev/docker_ghost_demo/Ghost/:/var/lib/ghost -d ghost 

``` 
Lets break the above command down  

docker run creates a container from the ghost image which is the last parameter and gives it a name blog as specified by the --name  
-p external_port:internal_to_container_port exposes port 2368 of the container to the port 2368 of the host running docker  
Ghost runs on 2368 by default that is  the reason why we do this mapping  

-v mounts an a location on your host file system to a location inside a container in this case we have mounted. 
 
```
  /home/max/dev/docker_ghost_demo/Ghost/ -> /var/lib/ghost
```

This was necessary so that we have access to ghost configuration file and themes from outside the container. 

-d runs the container in a detached mode 

To check if all went well you can run docker ps command 

```
CONTAINER ID        IMAGE               COMMAND                CREATED              STATUS              PORTS                    NAMES
98fdcfa4f46f        ghost:latest        "/entrypoint.sh npm    About a minute ago   Up About a minute   0.0.0.0:2368->2368/tcp   blog                
```

Now if you navigate to localhost:2368 you should see shiny new Ghost installation. 


### Installing MySQL(MariaDb) Docker image  

The principle here is the same as to what we did with the ghost blog container. 

Pull the image and run it with parameters, you just run without pulling docker will just download the image for you automatically.  

```
docker run --name ghost-mysql -v /home/max/dev/docker_ghost_demo/MariaDb/:/var/lib/mysql -e MYSQL_ROOT_PASSWORD="password" -d -p 3306:3306 mariadb
```

```
--name ghost-mysql: The name of my container 
```

Then we mount a local folder to be used as  /var/lib/mysql inside the container, this way our data is not destroyed should the container be removed. 
In other cases you can also create a data container, but is beyond the scope of this post.   

``` 
 -v /home/max/dev/docker_ghost_demo/MariaDb/: /var/lib/mysql 
```

```
-e MYSQL_ROOT_PASSWORD="password"
``` 

Sets up mysql root password of the container, replace with something more secure ;) 

-d : detached mode 
-p : Port mapping  

To verify that all went well you can run docker ps, you should see 2 running containers now. 

```
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                    NAMES
877b1c6e5ae9        mariadb:latest      "/docker-entrypoint.   6 minutes ago       Up 6 minutes        0.0.0.0:3306->3306/tcp   ghost-mysql         
98fdcfa4f46f        ghost:latest        "/entrypoint.sh npm    17 minutes ago      Up 17 minutes       0.0.0.0:2368->2368/tcp   blog  
```

### Configuring Ghost to talk to Mysql database.   

By default Ghost installation uses sqllite database, although this suffices in most cases i still prefer the stability and robustness of MySQL. 

Since we have a mysql container running lets configure Ghost to talk to the mysql container.  
Before we proceed we will need to create the database and the user for the ghost blog. 

You can do this by logging into your mysql container 
You will need mysql client installed ( sudo apt-get install mysql-client), if you dont have one.   

```
mysql -uroot -p -h127.0.0.1
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.5.5-10.1.11-MariaDB-1~jessie mariadb.org binary distribution
```

 
 Create the database, blog_user and grant all rights to the user. 
 
```
 mysql> create database blog; 
Query OK, 1 row affected (0.00 sec)

 mysql> create user 'blog_user'@'%' identified by 'blog_password'; 
Query OK, 0 rows affected (0.00 sec)

mysql> grant all privileges on *.* to 'blog_user'@'%'; 
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges; 
Query OK, 0 rows affected (0.00 sec) 

``` 




### Linking Ghost and Mysql Containers.  

we will need to recreate the container with the link. 

So stop the container with docker stop 

```
docker stop blog
docker rm blog 
```

Then we recreate the container specifying the link   

```
docker run --name blog -p 2368:2368 -v /home/max/dev/docker_ghost_demo/Ghost/:/var/lib/ghost -d --link ghost-mysql:mysql ghost 
```

This is the same as we did above the only difference is now we are linking two containers to each other using the 
--link parameter (ghost-mysql: "alias")  
The alias is the name that we shall use inside the ghost docker container to refer to the ghost-mysql database so we simply call it MySQL 

To prove that this is working as advertised we can log into the blog container and see if we have that link to MySQL 

```
docker exec -it blog /bin/bash 
root@4a53d36261bb:/usr/src/ghost# cat /etc/hosts
172.17.0.82	4a53d36261bb
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.81	mysql 877b1c6e5ae9 ghost-mysql 

```

To login into the running container you can use docker exec and the entry point inside the container, in our case bash is installed by default inside the ghost container so lets use that  
Checking hosts file shows our link is pointing to 1172.17.0.81 and is given the name of MySQL as we specified above.  

You can even ping that host 

```
ping mysql 
PING mysql (172.17.0.81): 56 data bytes
64 bytes from 172.17.0.81: icmp_seq=0 ttl=64 time=0.155 ms
64 bytes from 172.17.0.81: icmp_seq=1 ttl=64 time=0.088 ms
64 bytes from 172.17.0.81: icmp_seq=2 ttl=64 time=0.081 ms

```

Now we can edit Ghost configuration file to make it talk to the database we just created. 


```
 development: {
        // The url to use when providing links to the site, E.g. in RSS and email.
        // Change this to your Ghost blog's published URL.
        url: 'http://kimambo.de',

        // Example mail config
        // Visit http://support.ghost.org/mail for instructions
        // ```
        mail: {},

        // #### Database
        // Ghost supports sqlite3 (default), MySQL & PostgreSQL
        database: {
            client: 'mysql',
            connection: {
                host: 'mysql',
                user: 'blog_user',
                password: 'blog_password',
                database: 'blog',
                charset: 'utf8'

            },
        },
        // #### Server
        // Can be host & port (default), or socket
        server: {
            // Host to be passed to node's `net.Server#listen()`
            host: '0.0.0.0',
            // Port to be passed to node's `net.Server#listen()`, for iisnode set this to `process.env.PORT`
            port: '2368'
        },
        // #### Paths
        // Specify where your content directory lives
        paths: {
            contentPath: path.join(process.env.GHOST_CONTENT, '/')
        }
    }
```
In the database section we specify the connection information, note that host is specified as simply mysql this will be the link that we will create between the mysql container and ghost blog container, you cant just use localhost here as it will point to the localhost of the container itself and since Docker IP's are dynamic we have to use the link to make 2 containers talk to each other.
  
Once you start the blog again you will see new table structure that was automatically created by Ghost. 
Start the blog container using **docker start blog**


```
show tables ; 
+------------------------+
| Tables_in_blog         |
+------------------------+
| accesstokens           |
| app_fields             |
| app_settings           |
| apps                   |
| client_trusted_domains |
| clients                |
| permissions            |
| permissions_apps       |
| permissions_roles      |
| permissions_users      |
| posts                  |
| posts_tags             |
| refreshtokens          |
| roles                  |
| roles_users            |
| settings               |
| tags                   |
| users                  |
+------------------------+

```


### Nginx Proxy configuration  

The last piece of our puzzle is having nginx infront of this installation 
We need to make a few folders to store nginx configuration 

```
 mkdir -p /home/max/dev/docker_ghost_demo/Nginx/{logs,certs,sites-enabled}
``` 

```
docker run -v /home/max/dev/docker_ghost_demo/Nginx/sites-enabled/:/etc/nginx/conf.d/ -v /home/max/dev/docker_ghost_demo/Nginx/certs/:/etc/nginx/certs -v /home/max/dev/docker_ghost_demo/Nginx/logs/:/var/log/nginx --name nginxserver -p 80:80 -d --link blog:blog nginx

```

Here we mount 3 volumes to be used by Nginx and we link blog container to nginx container. 
/etc/nginx/conf.d/ contains the configuration of all the sites that you want to enable.   
For the config file to be picked up by Nginx it should have the ending of conf. 
We shall name our file as _localhost.conf_
Here is the sample localhost.conf file.

```
server {
  listen 0.0.0.0:80;
  server_name localhost; 
  access_log /var/log/nginx/kimambo.de.log; 

  location / {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header HOST $http_host;
      proxy_set_header X-NginX-Proxy true;

      proxy_pass http://blog:2368;
      proxy_redirect off;
  }
client_max_body_size 10M; # Allow 10 MB files to be uploaded
}

``` 

Restart docker container once the config file is in place and you can access your blog via port 80 on localhost or the IP of the machine where you did this setup. 

### Troubleshooting 

The following commands might be handy to see whats going on withing the container  

Getting the list of running containers 

docker ps  

Get the list of all the containers on the system regardless of status 

docker ps -a 

Get the logs from a running container 

docker logs {container_id}  

If you still have issues please post a comment below. 
