+++
title = "Notes on Docker"
author = ["Shreyas Ragavan"]
date = 2019-08-17T15:31:00-06:00
lastmod = 2019-11-08T22:57:05-07:00
tags = ["Docker", "Data-Science"]
categories = ["Docker", "DataScience"]
draft = false
linktitle = "Docker - notes"
toc = true
[menu.docs]
  identifier = "notes-on-docker"
  weight = 2001
+++

Docker is a fascinating concept that could be potentially useful in many ways, especially in Data science, and making reproducible workflows / environments. There are [several](https://towardsdatascience.com/learn-enough-docker-to-be-useful-b7ba70caeb4b) [articles](https://towardsdatascience.com/docker-for-data-scientists-5732501f0ba4) which have great introductions and examples of using docker in data science

This is an evolving summary of my exploration with Docker. It should prove to be a handy refresher of commands and concepts.


## <span class="org-todo todo TODO">TODO</span> What is Docker {#what-is-docker}

A brief summary of what Docker is all about.

1.  The main idea: disposable buckets of code that can do a specific task and either exit or run indefinitely.
    1.  The task / purpose of the container could even be a single command. Like `pwd`, which is piped into another container.
    2.  In a way this is an extension of the Unix philosophy of small tools that can do a single task well (i.e reliably).
2.  These buckets of code can be connected with each other and also stacked on top of each other to form a pipeline.
3.  These buckets of code are complete libraries
4.  The buckets consist of images which can be launched as containers.
5.  Docker images are stored in a registry. There are a number of registries, of which dockerhub is popular.

These schematics provide a good refresher of the core concept of Docker:

{{< figure src="/ox-hugo/Container-vs-vm.png" caption="Figure 1: Container Vs VM" >}}

{{< figure src="/ox-hugo/engine-components-flow.png" caption="Figure 2: Docker Engine components" >}}


### Dive into Docker {#dive-into-docker}

This is an excellent course run by Nick Janatakis ([link](https://diveintodocker.com/?r=devto)), which enabled me to tie together various bits and pieces of knowledge I had about Docker. I would recommend this course for anybody starting out with Docker. A lot of the notes in this document were gathered while going through the course.


#### Biggest wins of Docker {#biggest-wins-of-docker}

-   isolate and manage applications.
-   eg:  12 apps with 12 dependency sets.
-   VM : waste of resources.
-   Vagrant : lets you manage VM's on the command line (including Docker)
    -   Disk space occupied for each app is very high.
    -   Overhead of system boot up and restart / killing is high.
-   Docker can be used to manage common dependencies.
    -   Example of time frame: 2 seconds for loading 8 services.
    -   Spinning up an entire stack is very fast, compared to a VM.
-   Docker: portability of applications and dev environment.
-   Dozens of scenarios where something works for you but not for me.
-   New dev environments can be discouraging. With all the libraries and dependencies already installed, it is possible to become aggressive with the actual development and experimenting with new technology.
-   Multiple versions of a programming language can be installed within a single docker container.
-   Smaller Microservices that talk to each other are not always good, but Docker enables this in a streamlined manner.
-   LXC: raw linux containers. Existed long before docker.
    -   uses runC
    -   very complicated and brittle system.
    -   runs only on Linux.
    -   LXC's are still better than VM's for rapid build and deploy.
-   ANSIBLE: what files and tools should be on a server (very basic definition)


## Easy ways to get documentation help {#easy-ways-to-get-documentation-help}

-   Just typing in `docker` will provide a list of primary level commands that can be used.
-   For further flags, provide the primary command like `docker run --help`
-   The official documentation is a good resource.


## Definitions {#definitions}

1.  Image: Setup of the virtual computer.
2.  Container:  Instance of an image. Many containers can run with the same image.


## <span class="org-todo todo TODO">TODO</span> Running Emacs on Docker {#running-emacs-on-docker}

-   Note taken on <span class="timestamp-wrapper"><span class="timestamp">[2019-07-07 Sun 17:25] </span></span> <br />
    Matrix DS offers a viable alternative as a platform. However, a customised docker container with all my tools is a good way to reproduce my working environment and also share my work with the community.
-   Note taken on <span class="timestamp-wrapper"><span class="timestamp">[2019-07-06 Sat 17:54] </span></span> <br />
    This needs to be evaluated. Today I have a vague idea : set up a docker container combining Rocker + data science at the command line + Scimax together. A separate layer could also cater to shiny apps.

-   <https://www.christopherbiscardi.com/2014/10/17/emacs-in-docker/>
-   [Silex - github](https://github.com/Silex/docker-emacs) : Also contains references to other kinds of Emacs docker containers


## <span class="org-todo todo TODO">TODO</span> Good Online resources for Rocker {#good-online-resources-for-rocker}

-   Introducing Rocker: Docker for R | R-Bloggers
-   Rocker: Using R on Docker - A Hands-On Introduction - useR2015\_docker.Pdf
-   Jessie Frazelle's Blog: Using an R Container for Analytical Models
-   ROcker Images - Wiki Github
-   Introduction to Docker - Paper
-   [ ] Need to find a way to extract a bunch of links from the bookmark and directly available with org Mode.
-   [X] Play with Docker [link](https://training.play-with-docker.com)


## <span class="org-todo todo TODO">TODO</span> Introduction to Rocker - Technical paper [link](https://arxiv.org/pdf/1710.03675.pdf) {#introduction-to-rocker-technical-paper-link}


## Docker installation {#docker-installation}


### Note on Docker Toolbox versus Native apps {#note-on-docker-toolbox-versus-native-apps}

The native Docker application uses the type 1 hypervisor (hyperkit for Mac OS and hyper-V for Windows). `docker-machine` uses a virtualbox based hypervisor (type 2). This can also be specified while creating docker machines.

In general, the native applications have a better user experience and commands can be directly typed into the terminal. The native apps (on Windows/ Mac OS) are newer than the Docker toolbox, and are being actively developed by the Docker company to reach performance on par with the original virtualbox based Docker Toolbox approach.

Note that any performance lag depends on the application and as a thumb rule it may be better to start off with the native applications and switch to the toolbox when required.


### Installing Docker on debian {#installing-docker-on-debian}

-   Note taken on <span class="timestamp-wrapper"><span class="timestamp">[2019-08-05 Mon 22:18] </span></span> <br />
    Used these instructions to install docker on the VPS. The code was directly copied to a temporary org file on an Emacsclient on the VPS and some of the commands were run on Eshell.

The docker repository has to be added first for being able to install docker. Detailed instructions are available at <https://docs.docker.com/install/linux/docker-ce/debian/>.

A package is also available, and is probably the easiest method to install. Choose the appropriate version at:  <https://download.docker.com/linux/debian/dists/>

Manual version without using the package:

Adding Docker's official GPG key:

```sh
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
```

Searching that the key has been installed:

```sh
sudo apt-key fingerprint 0EBFCD88
```

pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]

Adding the stable Docker repository:

```sh
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
```

Update the package lists and now search for docker-ce. It should be available since the repository has been added and the list updated.

```sh
sudo apt-get update
```

Installing docker and necessary components. Note that the manual recommends removing any older installations if they exist.

Note from the manual that different versions of docker can be installed by including `sudo apt-get install docker-ce=VERSION=abcd`. Therefore multiple versions can probably exist side by side.

```sh
sudo apt-get install docker-ce docker-compose docker-ce-cli containerd.io
```

Creating a docker group and adding this to the sudoers list will enable running docker commands without using root privileges (`sudo`). A logout will be necessary to have the changes take effect.

Note: Sometimes the `$USER` variable does not seem to work. This can be replaced with your actual user name.

```sh
sudo groupadd docker
sudo usermod -aG docker $USER
```

To configure docker to start on boot, enable it as a service. The need to do this depends on how frequently you use docker commands.

```sh
sudo systemctl start docker
```


### Installing Docker on Antergos / Arch Linux {#installing-docker-on-antergos-arch-linux}

Installation can be done via Pacman

```sh
sudo pacman -S docker
```

Enable and start docker service.

```sh
sudo systemctl enable docker
sudo systemctl start docker
```

Add docker to the user's group using `usermod`. After adding this, a log-out is necessary. Note that $USER can be replaced with the output of `whoami` in the shell if desired. If this step is not performed, each docker command will have to be executed with `Sudo` elevation.

```sh
sudo usermod -a -G docker $USER
```


### Installing Docker on Mac OS {#installing-docker-on-mac-os}

Docker can be downloaded as an app from the docker store :  <https://hub.docker.com/editions/community/docker-ce-desktop-mac>.

On the Mac, the docker app has to be launched run first, and this will create a docker icon in the menu bar indicating the status of the docker machine. This launches the docker daemon, and then commands can be directly entered into the terminal.

Docker can also be installed using Brew:

```shell
brew cask install docker
```

This created an app in the Applications folder which has to be launched. However, it seems additional components are required to run Docker from the command Line. These are available via brew.

```shell
brew install docker-compose docker-machine
```


### Checking the installation {#checking-the-installation}

```shell
docker info
```

Trying the hello world container as an additional check. Note the steps listed in the output, which is the typical process.

```shell
cd ~/docker-test
docker run hello-world
```

Checking `docker-compose` version.

```sh
docker-compose --version
```


## General notes on containers and images {#general-notes-on-containers-and-images}

-   images contain the entire filesystem and parameters needed to run the application.
-   When an image is run, a container is created.
-   containers are generally immutable and changes do not linger
-   One image can spawn any number of containers, simultaneously. Each container will be separate.


## Default location of images {#default-location-of-images}

By default, on Antergos (Linux), the images are stored at `/var/lib/docker/`

```shell
sudo ls -al /var/lib/docker
```


## Docker version and info {#docker-version-and-info}

```shell
docker --version
docker info
docker version
```


## Listing Docker containers and images {#listing-docker-containers-and-images}

List Docker Images

```shell
docker image ls
```

List running Docker Containers

```shell
docker container ls
```

List all docker containers (running and Stopped)

```shell
docker container ls -a
```

Obtain only container ID's  (All). This is useful to extract the container number alone. The `q` argument stands for quiet.

```shell
docker container ls -aq
```


## Getting started {#getting-started}

[Ropenscilabs](http://ropenscilabs.github.io/r-docker-tutorial) has a basic introduction to Docker, and the [Docker documentation](https://docs.docker.com/get-started/part2/) is also a good place to start. A rocker specific introduction is available [here](https://github.com/BillMills/Rocker-tutorial/).

If a local image is not found, docker will try to search and download the image from docker hub.

It is better to create a folder wherein the docker container will reside.

```shell
mkdir ~/docker-test/
cd ~/docker-test
docker --rm -p 8787:8787 rocker/tidyverse
```

The `--rm` flag indicates the container will be deleted when the container is quite. The `-p` flag denotes using a particular port.
iner a
Note that the interim messages and download progress are not shown in eshell.

[Different rocker images](https://www.rocker-project.org/images/) are available, depending on the need to be served.


## Attaching shells `-t` and Interactive containers `-i` {#attaching-shells-t-and-interactive-containers-i}

Example to run an ubuntu container and run bash interactively, by attaching a terminal to the container. This will login to Ubuntu and start bash.

An alternative option is to use alpine linux, which is a much smaller download.

```shell
docker run -t -i ubuntu /bin/bash
```

```sh
docker run -ti alpine /bin/bash
```


## Running a detached container {#running-a-detached-container}

-   use the `-d` flag

<!--listend-->

```shell
docker container ls -al
docker run -d ubuntu
```


## Build process of a docker image {#build-process-of-a-docker-image}

-   `docker commit` : used to commit changes to a new image layer. This is a manual process. Commit has little place in the real world. Dockerfile is superior.
-   Dockerfile : blue print or recipe for creating a docker image. Each actionable step becomes a separate layer.

Docker image : result of stacking up individual layers. Only the parts or layers that have changed are downloaded for a newer version of a specific image.

Scratch image: docker image with no base operating system


## Working with dockerfiles {#working-with-dockerfiles}

-   sample or reference docker files can be saved as "dockerfile.finished" or with some other useful extension.
-   Dockerfiles are read top to bottom.
-   the first non-comment instruction should be `FROM`
    -   `FROM` allows you import a docker image.
-   `RUN` : basically executes the specified commands
-   `WORKDIR` : setting the desired working directory. This can be set or used multiple times in the same docker file.


## Running Rstudio via rocker/tidyverse {#running-rstudio-via-rocker-tidyverse}

Running a self-hosted Rstudio IDE is actually simple using rocker containers. I've chosen rocker/tidyverse because it would be a good base to build off on.

The initiation has to include a password for the default user, which is rstudio.

The below command will start a docker container that is served at port 8787 on the host.

The rocker/tidyverse image can be pulled first

```shell
docker image pull rocker/tidyverse
```

```shell
docker run -itd --name tidy1 -v $(pwd):/home/rstudio --user rstudio -p 8787:8787 -e PASSWORD=ABCD rocker/tidyverse
```


## docker image rm <repo id> or <tag> {#docker-image-rm-repo-id-or-tag}


## docker writes out a config file ~/.docker/config.json {#docker-writes-out-a-config-file-dot-docker-config-dot-json}

This will contain auth details.


## docker image before pushing should be tagged with username {#docker-image-before-pushing-should-be-tagged-with-username}


## appears there is no way to rename a tag {#appears-there-is-no-way-to-rename-a-tag}

One solution is to create a new container based on an existing tag, and rename it as desired.

Say there is an existing image named shrysr/web1, and I want to remove 'shrysr' from the repository name. The method would be:

```text
docker image build tag shrysr/web1 web1:latest
```

After this the old repository shrysr/web1 can be deleted.


## Adding flags while running a container <code>[0/1]</code> {#adding-flags-while-running-a-container}

`docker container run` allows access to management commands in docker.

-   `-i` : interactive
-   `-t` : terminal, on which bash can be initiated with `/bin/bash`
-   `-t` : terminal
-   `-p` : port
-   `-e` : environment variable
-   `--name` : naming the container.
-   `-d` : running a detached container in the background.
-   `--rm` : remove/delete a container once it is stopped.
-   `--restart` `on-failure` is an option. The `--rm` flag has to be removed. This will re-start if say, the docker daemon itself is restarted. It will not restart if manually stopped using `docker container stop`.
-   `-v` : attach a volume from the local host.

<!--listend-->

-   along with `p` - 2 ports are to be provided :- host:container
-   each port can be bound only once. It is also possible to specify a single port number, which will correspond to the hosts port number.

<!--listend-->

```shell
docker container run -it -p 5000:5000 web1

# Alternately, specifying a different port
# docker container run -it -p 5001 --name web01_c2 web1
```

-   container names have to be unique.
-   `-e` can take single / multiple arguments.
-   another flag can also take in a file containing environment variables.
-   [ ] presume that this means that the port number has to be remembered when running a container? Is this the way to run multiple docker apps, or even shin apps?
-   `interactive` as in : providing commands and interacting with the container.
-   depending on the image size, not usin the `--rm` flag could eat up disk space.

Example of running a container with several flags:

```shell
# navigate to the directory where the flask app exists, since the paths in the docker file are relative to the current directory.

# the web1 at the end, after the -e argument is the name of the image that is to be used. A container is an instance of an image.

#  This command is essentially running an image named web1, in a container named web1_cont on the port 5000 ( host :docker container) and passing in the argument specifying the name of a file containing the flask app. This is similar to calling a shiny app.

# cd
docker container run -it -p 5000:5000 --rm --name web1_cont  -d -e FLASK_APP=app.py web1

# Example of using restart on failire
docker container run -it -p 5001 --name web1_c2 -d --restart on-failure -e FLASK_APP=app.py web1
```


## Finding the docker IP {#finding-the-docker-ip}

This is needed while using the docker toolbox. I am yet to figure this out or use it.

```shell
docker-machine ip web1
```


## Viewing the log of a container {#viewing-the-log-of-a-container}

Logs of a stopped container can also be viewed.

```shell
docker container logs web1
```

-   tail the log file of a container using the `-f` flag. This is similar to the unix concept of tailing log files.
-   For exmaple, a window configuration could be devised in tmux tailing different files for real time updates.

<!--listend-->

```shell
docker container logs -f web1
```


## Attaching a volume with the -v {#attaching-a-volume-with-the-v}

```shell
docker container run -it -p 5002 --name web_c3 -e FLASK_DEBUG=1 -v $PWD:/app web1
```


## Connecting to an existing container {#connecting-to-an-existing-container}

Presuming that this container is connected to a volume with the -v flag.

```shell
# Obtain the container name or the ID
docker container ls
docker container exec -it web1 sh

# creating a temporary file
docker container exec -it web1 touch "t1.txt"

```

If running docker on a linux machine - any files created, on a linux system will be owned by the root user. This can be remedied by using the `--user` flag. However, this requires the arguments of the user and the group that the user belongs in.

Environment variables can be used for making this generic argument :
`"$(id -u):$(id -g)"` which translates to "current user id: current user group".

```shell
docker container exec -it --user "$(id -u):$(id -g)" web_c6 touch "t2.txt"
```


## Linking multiple docker containers {#linking-multiple-docker-containers}

Simplified definition: 2 computers need to be on the same network to communicate with each other, irrespective of being virtual or physical.

-   [ ] 0.0.0.0" : is a special address that any computer on your network can connect to.

Docker creates some networks by default. These can be found with

```shell
docker network ls
```

To inspect a specific docker network:

```shell
docker network inspect bridge
```

The bridge network will contain the details of the currently running containers.

To find the network details:  `ifconfig` on linux/ Mac OS and `ipconfig` on Windows. This command can be `exec` on a container. The `-it` flags are not required, since we just want the output.

```shell
docker container ps
```

172.17.0.4- web2
3 - redis\_r

```shell
docker container exec redis_r2 ping 172.17.0.4
```

The container's IP address can also be found by inspecting the container.

```shell
docker container inspect redis_r2
```

In particular, the redis container's IP address can be found from /etc/hosts

```shell
docker exec redis_r2 cat /etc/hosts
```

| ff02::1    | ip6-allnodes   |
|------------|----------------|
| ff02::2    | ip6-allrouters |
| 172.17.0.3 | 561402b7afe8   |

However for linking these containers, the IP addresses need not be hard coded, and extracted each time in the above method. This is resolved by allowing an automatic DNS set by docker.


### Creating a network and adding containers to it {#creating-a-network-and-adding-containers-to-it}

This creates a network named firstnetwork.

```shell
docker network create --driver bridge firstnetwork
# alternately use -d instead of --driver
# Access docs with:  docker network create --help
```

Verify that it is listed in the docker network list.

```shell
docker network ls
```

Running a container on a specified network means an additional `-net` flag, which will specify in this case firstnetwork.  To switch a container to a network, it has to be stopped and run again.

```shell
docker container run -itd --name web2 --rm --net firstnetwork -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 -v $PWD:/app web2
```

Running the redis container, with data persisting in the container.

```shell
docker container run -itd --name redis --rm --net firstnetwork -p 6379:6379 -v web2_redis:/data redis:latest
```

```shell
docker network inspect firstnetwork
```

By assigning both containers to the same network - we can ping them and execute commands from one on the other by using the hostname, instead of a cumbersome ip address that has be to verified each time.


#### Notes on Redis <code>[0/3]</code> {#notes-on-redis}

the `redis-cli` command can be used to access the command line on redis images. This does not appear to work on eshell. A normal terminal had to be used.

-   [ ] KEYS \* - command used to list the counter in the redis cli
-   [ ] INCRBY <variable> <value> : used to increment the redis containers counter.

-   <span class="org-todo todo TODO">TODO</span>  check out the modified app.py


## Persisting data in containers {#persisting-data-in-containers}

The flask app requires Redis to run. Therefore it is better to run the redis container first and have that available for the flask app for the first run.

Be cautious while storing data on containers. They are better left stateless and portable.


### <span class="org-todo todo TODO">TODO</span> Quick check: containers are running on the network {#quick-check-containers-are-running-on-the-network}

Checking that the containers are running in the firstnetwork. Obtain the container ID's from

```shell
docker container ps
```

Verify either the names or the container ID's from the `inspect` of firstnetwork.

```shell
docker network inspect firstnetwork
```


### `docker volume` : Named volumes to persist data in containers {#docker-volume-named-volumes-to-persist-data-in-containers}

A named volume can be created by

```shell
docker volume create web2_redis
```

This volume can be inspected and listed using the usual commands:

```shell
docker volume ls  >> ~/temp/docker-output.txt
cat >> ~/temp/docker-output.txt << EOF
# =================================================== Volume inspect
EOF
docker volume inspect web2_redis >> ~/temp/docker-output.txt
cat >> ~/temp/docker-output.txt << EOF
# =================================================== redis container inspect
EOF
docker container inspect redis >> ~/temp/docker-output.txt
```


## <span class="org-todo todo TODO">TODO</span> Accessing the docker volume {#accessing-the-docker-volume}

One way to do this is to run a shell on the redis container and and inspect the contents.

```shell
docker container exec -it redis redis-cli sh
```


## Sharing data between containers `--volume-from` <container name> {#sharing-data-between-containers-volume-from-container-name}

Recap on types of volumes seen so far:

1.  Named volume - to create a docker volume. Here docker will take care of the logistics of storing that data.
2.  `-v` : using the hosts file to mount a volume
3.  `--volume-from` : add this argument while running a container.


## Sample docker file {#sample-docker-file}

Steps:

1.  Create a directory, and set it as the working directory.
2.  Copy a text file containing library lists that have to be installed.
3.  Run pip to install sourcing from the said list.
4.  Copy the entire folder contents into the current working directory.
5.  Apply a label to the image.
6.  Attach a volume - specifying the public folder. Notice the volume command which references the folder public (containing a simple css file `main.css`, which is present in the same working folder, as is app.py, which was already copied in.
7.  Finally a _command_ is given, which is run after building the image. The _run_ command executes commands while the image is being built.

<!--listend-->

```text
FROM python:2.7-alpine

RUN mkdir /app
WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

LABEL maintainer="Shreyas Ragavan <sr@eml.cc>" \
      version="1.0"

VOLUME ["/app/public"]

CMD flask run --host=0.0.0.0 --port=5000

```


## `.dockerignore` file {#dot-dockerignore-file}

1.  docker wil check whether dockerignore files exists. If so, the specified patterns will be ignored.
2.  create a file and add .dockerignore as the first line.
3.  All the contents in a specific folder, but include the folder .foo/\*
4.  All files with the extension `**/*.swp`
5.  Ignore all text files, except `special_me.txt`.

<!--listend-->

```text
.dockerignore

.git/
.foo/*
**/*.swp
**/*.txt
!special_me.txt
```


### Converting a blacklist to a whitelist {#converting-a-blacklist-to-a-whitelist}

A blacklist can be converted into a whitelist by including a `*` as the very first character in the file, before `.dockerignore`.

```text
*
.dockerignore

.git/
.foo/*
**/*.swp
**/*.txt
!special_me.txt
```
