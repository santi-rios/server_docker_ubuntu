
```bash
sudo docker-compose -f ~/docker/docker-compose.yml up -d
```

### get all commands
`docker`

### Information on your docker installation
`sudo docker info`

### Search for specific images/containers
-   **sudo docker search IMAGE_NAME** - Search for specific images/containers.

### Start/stop/restart one or more containers
-   **sudo docker start CONTAINER_NAME** - Start one or more containers.

-   **sudo docker stop CONTAINER_NAME** - Stop one or more containers.

-   **sudo docker restart CONTAINER_NAME** - Restart one or more containers.

### See running processes of a container
-   **sudo docker top CONTAINER_NAME** - See running processes of a container.

### Remove one or more containers
-   **sudo docker rm CONTAINER_NAME** - Remove one or more containers.

### Pull updated images from Docker Hub
-   **sudo docker pull CONTAINER_NAME** - Pull updated images from Docker Hub.

### See all defined docker networks
-   **sudo docker network ls** - See all defined docker networks.

### See all running containers.
-   **sudo docker ps -a** - See all running containers.

### See logs of a one, more, or all containers
-   **sudo docker logs CONTAINER_NAME** - See logs of a one, more, or all containers.

### Start/stop specific or all services defined in docker-compose.yml.
-   **sudo docker-compose up -d CONTAINER_NAME** - Start specific or all services defined in docker-compose.yml.

-   **sudo docker-compose down CONTAINER_NAME** - Stop specific or all services defined in docker-compose.yml.

```
sudo docker-compose -f ~/docker/docker-compose.yml up -d
```

The `-d` option daemonizes it in the background. Without it, you will see real-time logs, which is another way of making sure no errors are thrown. Press `Ctrl + C` to exit out of the real-time logs.


## docker cleanup

So at any time you can run the following clean up scripts and re-run your docker-compose as described above.
`sudo` `docker system prune`

`sudo` `docker image prune`

`sudo` `docker volume prune`

These commands will remove any stray containers, volumes, and images that are not running or are not associated with any containers.

``docker system prune -a -f``
- Ver bien que hace este

#to remove all log files  
**find /var/lib/docker/containers/ -type f -name “*.log” -delete**

`docker system prune --all --volumes --force`
- usar con cuidado
