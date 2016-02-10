##Docker Installation

as root do the following 

apt-get install apt-transport-https ca-certificates  
apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D  
touch /etc/apt/sources.list.d/docker.list  
nano /etc/apt/sources.list.d/docker.list  
add this to the docker.list   
deb https://apt.dockerproject.org/repo ubuntu-wily main  
apt-get update   
apt-get purge lxc-docker   
apt-cache policy docker-engine  
apt-get upgrade  
apt-get install docker-engine  
docker run -it ubuntu /bin/bash  

You can start the service via   
sudo service docker start  
 

Check the status of docker engine  

```
service docker status -l
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Mi 2016-02-10 10:27:38 CET; 6min ago
     Docs: https://docs.docker.com
 Main PID: 15690 (docker)
   CGroup: /system.slice/docker.service
           └─15690 /usr/bin/docker daemon -H fd://

```

We want to pull node server image  

``` 
docker pull node 
```

To see what images we have in the inventory  

```
root@el-mariachi:/home/max# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
node                latest              963838ccc9c1        12 hours ago        644.2 MB
ubuntu              latest              3876b81b5a81        3 weeks ago         187.9 MB
tutum/hello-world   latest              31e17b0746e4        8 weeks ago         17.79 MB

```

Lets create a mongoDB container 

first pull 

	docker pull mongo:latest  
	
Mongo requires  /data/db, to achieve this we can mount a host folder into the container  

```
max@el-mariachi:~$ mkdir  data 
max@el-mariachi:~$ pwd
/home/max
max@el-mariachi:~$ sudo docker run -t -i -v /home/max/data:/data -p 27017:27017 --name mongodb -d mongo
44b3a08ce6198110ab02d9bb9babb93480acbf83b6db015bf33e913164beee38
max@el-mariachi:~$ sudo docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
44b3a08ce619        mongo               "/entrypoint.sh mongo"   11 seconds ago      Up 9 seconds        27017/tcp           mongodb
```

We first created the data folder on the host  
Then you mount it via the -v options specifying source and target of the mount source_on_host:target_in_container  
 
To check if the process is running do  

	sudo docker ps 


Some interesting operations could be 
- Checking the logs of mongoDb 
 
```
  sudo docker logs mongodb
2016-02-10T10:10:24.085+0000 I CONTROL  [initandlisten] MongoDB starting : pid=1 port=27017 dbpath=/data/db 64-bit host=44b3a08ce619
2016-02-10T10:10:24.085+0000 I CONTROL  [initandlisten] db version v3.2.1
2016-02-10T10:10:24.085+0000 I CONTROL  [initandlisten] git version: a14d55980c2cdc565d4704a7e3ad37e4e535c1b2
2016-02-10T10:10:24.085+0000 I CONTROL  [initandlisten] OpenSSL version: OpenSSL 1.0.1e 11 Feb 2013
2016-02-10T10:10:24.085+0000 I CONTROL  [initandlisten] allocator: tcmalloc
2016-02-10T10:10:24.085+0000 I CONTROL  [initandlisten] modules: none

```


- Connecting to the Shell of the container  
 
 
 ``` 
 sudo docker exec -it mongodb bash 
	root@44b3a08ce619:/# 
```


use the following to start/ stop the container. 

	sudo docker stop mongodb
	sudo docker start mongodb







