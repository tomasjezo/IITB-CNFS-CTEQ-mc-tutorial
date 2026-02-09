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

## Setup for the MC tutorial

1. We first pull the `mc-tutorial-2026` image from the docker hub
   ```bash
   docker pull tomasjezo/mc-tutorial-2026
   ```

2. [Optional] We check whether we the right architecture was downloaded.
   ```bash
   docker image inspect tomasjezo/mc-tutorial-2026  --format '{{.Os}}/{{.Architecture}}'
   ```
   It should return
   ```
   linux/amd64
   ``` 
   on Linux, Windows and older Mac machines, and
   ```
   linux/arm64
   ```
   on Macs with M chipset.

3. Then we enter the directory we want to work in and create a container from the image
   ```bash
   docker run -itd -v ./:/home -w /home --name mc-tutorial tomasjezo/mc-tutorial-2026
   ``` 
   The `-it` flag runs the container with a terminal attached to it, `-d` in detached state.
   The `-v` flag mounts the current working directory under the `/home` path in the container and the `-w` flag changes the working directory to `/home`. `--name` makes the container available under the name `mc-tutorial`.

   The container now should appear when listing the containers. The command
   ```bash
   docker ps
   ```
   yields
   ```bash
   CONTAINER ID   IMAGE          COMMAND   CREATED          STATUS                      PORTS     NAMES
   <id>           <image_id>     "bash"    xy seconds ago   Up xy seconds                         mc-tutorial
   ```

4. Finally, we can execute any command in the container as follows
   ```bash
   docker exec mc-tutorial <COMMAND>
   ```
   If you want to run an interactive command like `python` make sure to use the `-it` flag
   ```bash
   docker exec -it mc-tutorial <COMMAND>
   ```

5. Alternatively it may be useful to setup an `alias`, if available in your terminal:
   ```bash
   alias dexec="docker exec -it mc-tutorial"
   ```
   The command
   ```bash 
   dexec <COMMAND>
   ```
   will then be automatically automatically substituted for
   ```bash 
   docker exec -it mc-tutorial <COMMAND>
   ```
   This is how we will use the container throughout the tutorial.

Note that instead of executing the commands from the host system using `dexec` you can also just "attach" the container with
```bash
docker exec -it mc-tutorial bash
```
and type all the commands in the tutorial directly in the terminal inside the container without `dexec`.

## Running Python with the `mc-tutorial` container 

- run `python` interpreter terminal
   ```
   dexec python
   ```
   type command and execute them there. The `-it` flag ensures the interactive mode is invoked, otherwise the command just exits.

- save the code to a source file `<fname>.py` and run
   ```
   dexec python <fname>.py
   ``` 
   The `-it` flag is not necessary, but won't hurt. 

## Compiling `Pythia8` programs with the `mc-tutorial` container 

Here we assume that the directory you work in is the same as the working directory used during the creation of the container. Otherwise, make use of the `-w` flag to tell docker the current working directory.

1. Get the `Makefile` from the container 
   ```
   dexec cat /usr/local/share/Pythia8/examples/Makefile > Makefile
   dexec cat /usr/local/share/Pythia8/examples/Makefile.inc > Makefile.inc
   ```
2. a. If you want your progam to be linked against `libpythia8` then you can simply name your source file `mymainNN.cc` where NN is between 01 and 99 and run
   ```
   dexec make mymainNN
   ```
   b. If also need linking against `HepMC3` then name your source code as you wish, for example `mysource.cc`. Then you need to modify the `Makefile` by replacing the line 91
   ```
   main131 main132 main133 main134 main135: $(PYTHIA) $$@.cc
   ```
   by 
   ```
   main131 main132 main133 main134 main135 mysource: $(PYTHIA) $$@.cc
   ```
   and run
   ```
   dexec make mysource
   ```
