# `Docker` notes

Here we describe our recommended way of using `Docker`.

## Hello world
- Pull image 
   ```bash
   docker pull hello-world
   ```
- Check it's there
   ```bash
   docker images
   ```
   This should print
   ```
   REPOSITORY                        TAG       IMAGE ID       CREATED         SIZE
   hello-world                       latest    ee301c921b8a   15 months ago   9.14kB
   ```
- Run it (turn it into a container with a name, see `--name` switch)  
  ```bash
  docker run --name hello-world-container hello-world
  ```
  This will show the hello world message from Docker.
  ```
   Hello from Docker!
   This message shows that your installation appears to be working correctly.

   To generate this message, Docker took the following steps:
    1. The Docker client contacted the Docker daemon.
    2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
       (arm64v8)
    3. The Docker daemon created a new container from that image which runs the
       executable that produces the output you are currently reading.
    4. The Docker daemon streamed that output to the Docker client, which sent it
       to your terminal.

   To try something more ambitious, you can run an Ubuntu container with:
   $ docker run -it ubuntu bash

   Share images, automate workflows, and more with a free Docker ID:
    https://hub.docker.com/

   For more examples and ideas, visit:
    https://docs.docker.com/get-started/
   ```
  Under the hood, it will instantiate a container with a name "hello-world-container" from an imaged called "hello-world"
 
- Listing all running docker containers can be done with
   ```bash
   docker ps
   ```
- But our hellow-world-container container ran and exited. To list also stopped containers, we add `-a` switch
   ```bash
   docker ps -a
   ```
- To start and stop the container we can do
   ```bash
   # start a container
   docker start hello-world-container
   # stop a container
   docker stop hello-world-contianer
   ```
- To delete a container run
   ```bash
   # delete a container
   docker rm hello-world-container
   ```

## First useful example: a basic linux container
- Pull image 
   ```bash
   docker pull debian:12
   ```
- Instantiate the container
  ```bash
  docker run -itd --name linux debian:12
  ```
- Attach to it
  ```bash
  docker exec -it linux bash
  ```
  `-it` flag is essential, links your terminal to the terminal in the container (`i` interactive, `t` attach pseudo terminal), otherwise the command executes and returns

## Warning: Potential data loss!
Every time you execute `docker run` a container is created from an image, if you make any modifications to the container these modifications are not automatically transferred to the image. So if you delete the container, all your modifications will be lost. Actually, many `Docker` tutorials suggest executing the `run` command with the `--rm` flag which instantiates the container, runs the dedicated task and deletes it as soon as it is finished. I do not recommend to use the `--rm` flag in this tutorial, unless you know what you are doing. 
If you want to preserve modifications, you need to keep it alive. Or turn the container back to an image with the `docker commit` command.

## Access data from the Host
You can mount a directory on the Host system in the container at the point of instantiation:
- Instantiate the container with the current directory mounted
   ```bash
   docker run -itd -v ./:/home -w /home --name linux debian:12
   ```
  `-w /home` means the default working directory will be `/home`
- Attach to it
   ```bash
   docker exec -it linux bash
   ```
