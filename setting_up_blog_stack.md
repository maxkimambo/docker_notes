Troubleshooting 

Incase you get 

docker-compose up 
ERROR: client and server don't have same version (client : 1.21, server: 1.18)


```
export COMPOSE_API_VERSION=1.18
```

Add user to sudoers group 

sudo adduser <username> sudo

or 

sudo usermod -a -G sudo <username>
