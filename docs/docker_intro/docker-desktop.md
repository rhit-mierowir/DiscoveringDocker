# Docker Desktop

The guide is pushing the use of docker desktop to visualize what is happening. I am finding it mildly challenging to install.

I was following the guidance in the main file, but there seemed to be an instruction that it missed when constructing the container. It was encapsulated in a popup on the right side of docker desktop titled **How do I run a container?**. It suggested running the following commands to run the container. I fear that these instructions won't give me much insight into the actual cmd interface.

``` bash
git clone https://github.com/docker/welcome-to-docker
cd welcome-to-docker
docker build -t welcome-to-docker .
```

This all generates an *image* holding all of this configuration that can then be run as a *container*.

Next: https://docs.docker.com/get-started/introduction/develop-with-containers/