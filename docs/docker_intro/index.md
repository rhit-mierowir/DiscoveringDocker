# Docker Introduction & Concepts

This work was derrived from [ the docker documentation, here](https://docs.docker.com/get-started/introduction/).

Any work for this will be in the `docker_intro` folder.

The following sections are based on the lessons from the lessons.

## Get Docker Desktop

The following code was used to create the welcome-to-docker image, which was then deployed as a server via docker desktop.

``` bash
git clone https://github.com/docker/welcome-to-docker
cd welcome-to-docker
docker build -t welcome-to-docker .
```

This all generates an *image* holding all of this configuration that can then be run as a *container*.

## Develop With Containers

To deploy the project, I ran the following code. I think this spins up multiple images.

```  bash
git clone https://github.com/docker/getting-started-todo-app
cd getting-started-todo-app
docker compose watch
```

This automatically installed Node, MySQL and other dependencies with almost zero effort.

Changes to the running software were ==performed as quickly as the files were saved== because docker automatically watches the involved files for changes and the files are shared in a containerized environment.

## Build and push your first image

I don't think that this is something we will be using, so I mostly just read through it. 

`dockerfile/image`
:   A text-based format to specify how to build an image.

Here are the commands that we would need to run:

``` bash
git clone https://github.com/docker/getting-started-todo-app
docker build -t <DOCKER_USERNAME>/getting-started-todo-app .
docker image ls  #This is to check that the image exists locally.
docker push <DOCKER_USERNAME>/getting-started-todo-app
```

Docker-Hub is the go-to repository for docker images, both official and those distributed by other individuals.

