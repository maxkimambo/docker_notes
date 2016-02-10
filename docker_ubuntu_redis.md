### Redis   

The are two ways of getting a redis container 

- Download the docker image   
	in this case you would need to run 
	``` 
	docker run  -d --name redis -p 6379:6379 redis 
	``` 
	
- Build it from the ubuntu base image 

	This can be achieved by writing a docker file that looks like this. 
	
	
```
# Set the base image to Ubuntu
FROM        ubuntu:14.04

# File Author / Maintainer
MAINTAINER Max Kimambo

# Update the repository and install Redis Server
RUN         apt-get update && apt-get install -y redis-server

# Expose Redis port 6379
EXPOSE      6379

# Run Redis Server
ENTRYPOINT  ["/usr/bin/redis-server"]

``` 

