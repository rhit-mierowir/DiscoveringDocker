# Running Containers

## Publishing and Exposing Ports

Docker containers generally run isolated from the rest of the system, but sometimes you want to interact with what they are publishing on their ports or expose them from ports on the host computer. This is where publishing ports comes in. 

In Dockerfiles, there is an `EXPOSE` command. This doesn't automatically publish the ports when the container is run, but simply makes the promice that this image is actually using this port and thus publishing it would be useful. 

To publish ports, use the `-p` command to specify the ports that you want to publish as shown below. The first commands specifies the exact port on the host computer to connect to, while the second lets the computer match it to an ephemeral port, and you can see the port chosen by running `docker ps`.

``` bash
docker run -d -p HOST_PORT:CONTAINER_PORT nginx
docker run -p CONTAINER_PORT nginx
```

If you use the `-P` flag, then all ports exposed with `EXPOSE` commands in the dockerfile will automatically be mapped to ephemeral ports.

``` bash
docker run -P nginx
```

## Overriding Defaults

Adding arguments to the `docker run` command allow it to automaticall override defaults of the container and customize it to your particular usecase

### Fixing port conflicts

If two services want to use the same port, simply map them to different ports when you are exposing them.

``` bash
docker run -d -p HOST_PORT:CONTAINER_PORT postgres
```

You can also create an internal network to allow you to connect to more ports or to enable docker containers to connect to eachother more easily. 

``` bash
docker network create NETWORK_NAME
docker run -d -p HOST_PORT:CONTAINER_PORT --network NETWORK_NAME postgres
```

### Changing Environment Variables

Set environment variables explicitly in the `docker run` command, or pass them in through a `.env` file.

``` bash
docker run -e foo=bar postgres env
docker run --env-file .env postgres env
```

### Set Resource Limits

To restrict the amount of memory or cpus a container is allowed to use, set `--memory` or `--cpus` flags.

``` bash
docker run -e POSTGRES_PASSWORD=secret --memory="512m" --cpus="0.5" postgres
```

## Persisting Container Data (Docker Volumes)

Since Containers are ephemeral and we often want data to be persisted between the running of containers, we can use `volumes` to store data in a permanent way independant of the container that is using it and independantly of the host file system. It is likely easier to manage volumes from the desktop, including ways to be able to look through the contents of volumes, but I shall remain on the commandline. 

Creating a Volume:

``` bash
docker volume create log-data
```

Managing volumes:

- `docker volume ls` - list all volumes
- `docker volume rm` <volume-name-or-id> - remove a volume (only works when the volume is not attached to any containers)
- `docker volume prune` - remove all unused (unattached) volumes


Creating A Container Using a Volume:

``` bash
docker run -d -p 80:80 -v log-data:/logs docker/welcome-to-docker
```

## Sharing Local Files With Containers

This can be done via `bind mounts`, and these are different from volumes because they link files in the container to files on your host whereas volumes mount to an empty directory managed by Docker. Technically, volumes are specific type of bind mount. If you want to persist information, use a volume. If you want to access a file on the host system, use a bind mount.

The `-v` flag is simpler and more convenient for basic volume or bind mount operations. If the host location doesn’t exist when using `-v` or `--volume`, a directory will be automatically created.

The `--mount` flag offers more advanced features and granular control, making it suitable for complex mount scenarios or production deployments. If you use `--mount` to bind-mount a file or directory that doesn't yet exist on the Docker host, the docker run command doesn't automatically create it for you but generates an error.

``` bash
docker run -v /HOST/PATH:/CONTAINER/PATH -it nginx
docker run --mount type=bind,source=/HOST/PATH,target=/CONTAINER/PATH,readonly nginx
```

You can also manage File permissions to determine if the mounted files can be edited, which are changes that would be reflected in the host. To grant read/write access, you can use the `:ro` flag (read-only) or `:rw` (read-write) with the `-v` or `--mount` flag during container creation

``` bash
docker run -v HOST-DIRECTORY:/CONTAINER-DIRECTORY:rw nginx
```

## Multi-Container Applications

If your application requires running multiple docker containers, you can build and run them individually, which is error-prone and slow, or you can simply use docker compose. This can be done simply with the following command, where the `--build` argument simply specifies to build the docker containers that you have constructed locally using DockerFiles. Not only does this simplify building and running all of the containers, but it also makes the application easier to manage, as all the associated containers are grouped, and their relationships are directly encoded in the `compose.yml` file.

``` bash
docker compose up -d --build
```