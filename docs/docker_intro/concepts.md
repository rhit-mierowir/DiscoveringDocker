# Concepts

This is where we are describing what I find in the `Docker Concept` section.

This will also go in roughly the order Basics -> Building -> Running.

## The Basics
### What is a container?

`Container`
:   An isolated process for each component of your app, with each component running in its own completely isolated environment.

Container Properties:

- **Self-contained.** Each container has no reliance on pre-installed dependencies on the host computer.
- **Isolated.** Increased security because each container has minimal influence on other containers.
- **Independent.** Each is independantly managed, so removing one doesn't impact others.
- **Portable.** - Each container works the same way everywhere.

Continers function as Virtual Machines, but they remove most of the unnecessary overhead, only running a particular process.

How to check for running containers:

``` bash
docker ps
```

Run an example:

``` bash
docker run -d -p 8080:80 docker/welcome-to-docker
```

This exposes port 80 of the container to port 8080 of the host device. Access via [http://localhost:8080/](http://localhost:8080/).

### What Is an Image?

`Docker Image`
: Everything required to run a container.
: a standardized package that includes all of the files, binaries, libraries, and configurations to run a container.

Images are immutable, so can't be changed, but you an add layers on top of docker images to add or remove files and functionality. This allows you to - for example - run python files on a pre-configured docker container image.

This image can be distributed to give everything needed to run the container on things like dockerHub.

If there are issues in an image (security vulnerability) you should replace them with a new and improved image, not change the image. 

Dockerfiles are somehow involved in this process.

There are two important principles of images:

1. Images are immutable. Once an image is created, it can't be modified. You can only make a new image or add changes on top of it.
2. Container images are composed of layers. Each layer ==represents a set of file system changes== that add, remove, or modify files.

Docker Hub is the default place to source images, but there are other sources that you can get them from.

``` bash
docker search docker/welcome-to-docker
docker pull docker/welcome-to-docker
docker image ls
docker image history docker/welcome-to-docker
```

This shows us looking on dockerhub for the container, pulling it to our local computer, validating that we have it, then checking the layers on that image.

### What is a Registry?

Registrys are repositories where images are stored, allowing you to share across teams. These are things like DockerHub. There is an API protocall (OCI spec) that is open source and available to be implemented by anyone. Without registrys Docker would work perfectly well, but each user would be forced to operate in isolation and wouldn't be able to share their images.

A `Registry` is slightly different from a `Repository`. 

`Registry`
: a centralized location that stores and manages container images

`Repository`
: a collection of related container images within a registry. 
: Think of it as a folder where you organize your images based on projects.

Each repository contains one or more container images, and a registry contains one or more repositories.


### What is Docker Compose?

While one container can run as many processes in it as you would like, this isn't good practice. Generally, you should set up or pull containers that do only one thing, and do it well.

To do more advanced tasks that require the use of multiple containers, you can use `Docker Compose` to coordinate the multiple `docker run` commands and start multiple containers, each with specific configuration. Otherwise, the task of running multiple containers is very error prone and cleanup is complicated. Docker Compose allows you to declare this structure in a yaml file that can simply be run once defined as many times as desired with a single command. To deploy containers, or adjust containers according to an edit, run `docker compose up`.

``` bash
docker compose up -d --build
docker compose down
docker compose down --volumes
```

The top `down` command just tares down the containers, but leaves the volumes created as persistant data, whereas the ones below it also tares down the volumes and deletes the related data.


