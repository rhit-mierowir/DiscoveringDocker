# Building Images

## Understanding Image Layers

Images are composed of layers, each of which is immutable once created. Each image contains a set of filesystem changes (additions, deletions, or modifications)

An example of layers in an image for an app:

    1. The first layer adds basic commands and a package manager, such as apt.
    2. The second layer installs a Python runtime and pip for dependency management.
    3. The third layer copies in an application’s specific requirements.txt file.
    4. The fourth layer installs that application’s specific dependencies.
    5. The fifth layer copies in the actual source code of the application.

This allows you to reuse layers, for instance, layers 1 & 2 can be reused for another python program. It also lets you extend images created by others.

### Stacking the layers

Layering is made possible by content-addressable storage and union filesystems. This allows you to reuse layers between images, but also to be able to run multiple containers from the same underlying image.

Edited Quote from lesson:

>    1. After each layer is downloaded, it is extracted into its own directory on the host filesystem.
>    2. When you run a container from an image, a union filesystem is created where layers are stacked on top of each other, creating a new and unified view.
>        1. A new layer is also createdfor each running container, allowing each container to edit its filesystem without impacting others.
>    3. When the container starts, its root directory is set to the location of this unified directory, using chroot.

`Base Image`
: Although all images can be used to add layers onto, some layers have little utility on their own and are instead intended to serve as the foundation for other images. Such images are refered to as a Base Image, like the ones seen in this example.

### Example

First, we use an existing debian container to create a node container, which we use to create our app container.

Here, `$` specifies that it is on be base terminal, `root@d8c5ca119fcd:/#` specifies that it is entered in the command line that appears in the container. 

``` bash

$ docker run --name=base-container -ti ubuntu
root@d8c5ca119fcd:/# apt update && apt install -y nodejs
root@d8c5ca119fcd:/# node -e 'console.log("Hello world!")'
$ docker container commit -m "Add node" base-container node-base
$ docker image history node-base
$ docker run node-base node -e "console.log('Hello again')"
$ docker rm -f base-container

```

Now we have created the node package from a base ubuntu container, and validated node was installed.

``` bash

$ docker run --name=app-container -ti node-base
root@d8c5ca119fcd:/# echo 'console.log("Hello from an app")' > app.js
root@d8c5ca119fcd:/# node app.js
$ docker container commit -c "CMD node app.js" -m "Add app" app-container sample-app
$ docker image history sample-app
$ docker run sample-app
$ docker rm -f app-container
```

Now, we have created a container that will automatically run our trivial little program and print out a message when run.

### Summary

Basically, you can imagine what happens to a docker container as sediment acrewing on the ocean floor. There is some base image, then people edit the container using the container-specific layer, and when that reaches a desired state they will commit it to a new image to lock in that state. 

Since this whole process was explained as file-system edits, I am not exactly sure how the commands passed into the container are preserved and re-run when the image is recreated, which was shown to happen when the app was finally constructed. 

## Writing a Dockerfile

A Dockerfile is a text file that allows you to transform an existing docker image into a new one by specifying the operations that must be done on it. 
This involves commands like specifying the working directory, copying files into the container, and running commands in the container. Then, the CMD option specifies what should be run when you start the container.

Each command corresponds to a new layer in your built image.

For example:

``` Dockerfile

FROM python:3.13
WORKDIR /usr/local/app

# Install the application dependencies
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

# Copy in the source code
COPY src ./src
EXPOSE 8080

# Setup an app user so the container doesn't run as the root user
RUN useradd app
USER app

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8080"]

```

Some of the most common instructions in a Dockerfile include: (Copied from course)

- FROM <image> - this specifies the base image that the build will extend.
- WORKDIR <path> - this instruction specifies the "working directory" or the path in the image where files will be copied and commands will be executed.
- COPY <host-path> <image-path> - this instruction tells the builder to copy files from the host and put them into the container image.
- RUN <command> - this instruction tells the builder to run the specified command.
- ENV <name> <value> - this instruction sets an environment variable that a running container will use.
- EXPOSE <port-number> - this instruction sets configuration on the image that indicates a port the image would like to expose.
- USER <user-or-uid> - this instruction sets the default user for all subsequent instructions.
- CMD ["<command>", "<arg1>"] - this instruction sets the default command a container using this image will run.

To read through all of the instructions or go into greater detail, check out the Dockerfile reference.

If you have an existing project, you can run `docker init` and it will analyze the project and attempt to quickly create a dockerfile, a compose.yaml, and a .dockerignore file to containerize your project.

Best practices make maintaining an image easier, use of the image quicker, and storage of the image and container more efficient.

[Docker File Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/){ .md-button .md-button--primary}

## Build, Tag, and Publish an Image

`Building Images`
: The process of building an image based on a Dockerfile

`Tagging Images`
: The process of giving an image a name, which also determines where the image can be distributed.

`Publishing Images`
: The process to distribute or share the newly created image using a container registry.

### Building Images

To turn a Dockerfile into an immutable image, run:

``` bash
docker build .
docker run <name of image>
```

The dot specifies the folder looked in for the Dockerfile and the files referanced in the Dockerfile. The images created will be be named as a sha hash. For a more interpretable name, use a tag.

### Adding Tags

The general format for a tag is as follows:

`[HOST[:PORT_NUMBER]/]PATH[:TAG]`

- `HOST`: The optional registry hostname where the image is located. If no host is specified, Docker's public registry at docker.io is used by default.
- `PORT_NUMBER`: The registry port number if a hostname is provided
- `PATH`: The path of the image, consisting of slash-separated components. For Docker Hub, the format follows [NAMESPACE/]REPOSITORY, where namespace is either a user's or organization's name. If no namespace is specified, library is used, which is the namespace for Docker Official Images.
- `TAG`: A custom, human-readable identifier that's typically used to identify different versions or variants of an image. If no tag is specified, latest is used by default.

Use `-t` or `--tag` flag to add a tag to a container you are building, or if you have the image already, use `docker image tag`

``` bash
docker build -t my-username/my-image .
docker image tag my-username/my-image another-username/another-image:v1
```

The same tag can refer to different images at different points in time, allowing for you to update the image without everyone having to change their Dockerfiles to match.

### Publishing Images

To push an image that you have already built and tagged to a registry, use `docker push`:

``` bash
docker push my-username/my-image
```

## Using the Build Cache

To avoid repeating steps that have already occoured when building a previous version of an image from a Dockerfile, the build cache stores the results of these steps so you can skip them for a faster complilation, however if any element that impacts one step changes, it invalidates that step and all those after it, which will be rerun. 

Each layer on an image will be reused by the Build Cache if no changes are detected on anything impacting that layer(command) or an earlier one.

## Multi-Stage Builds

While it works perfectly well to build everything in one container, that often leaves unnecessary bloat, as the complilation software or setup software is rarely used in the final application. This lets the docker container take up less space on the disk and also exposes it to fewer cybersecurity vulnerabilities because there is less to attack. 

This can be done by having two `FROM` commands that produce two different docker containers, only one of which will be the final build, which is by default the second one in the list, but there are commands to specify this explicitly. For this, you can start with a compilation-based image to compile the code, then copy this code from the first image into the second without having to include any information unnecessary to run the application. An example of this structure is shown below.

``` dockerfile
# Stage 1: Build Environment
FROM builder-image AS build-stage 
# Install build tools (e.g., Maven, Gradle)
# Copy source code
# Build commands (e.g., compile, package)

# Stage 2: Runtime environment
FROM runtime-image AS final-stage  
#  Copy application artifacts from the build stage (e.g., JAR file)
COPY --from=build-stage /path/in/build/stage /path/to/place/in/final/stage
# Define runtime configuration (e.g., CMD, ENTRYPOINT) 
```
