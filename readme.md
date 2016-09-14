# Deep Dive into Docker
These are course notes I'm taking to remember things.

# Table of Contents
- [Learning The Basics of Docker](#)
    - [Introduction to Docker](#)
    - [Containers Vs VirtualMachines](#)
    - [Docker Architecture](#)
    - [Docker Installation](#)
    - [Creating our First Image](#creating-our-first-image)
- [The Dockerfile, Builds and Network Configuration](#the-dockerfile-builds-and-network-configuration)
    - [Dockerfile Directives: User and Run](#dockerfile-directives-user-and-run)
    - [Dockerfile Directives: RUN Order of Execution](#dockerfile-directives-run-order-of-execution)
    - [Dockerfile Directives: ENV](#dockerfile-directives-end)
    - [Dockerfile Directives: CMD Vs Run](#dockerfile-directives-cmd-vs-run)
    - [Dockerfile Directives: ENTRYPOINT](#dockerfile-directives-entrypoint)
    - [Dockerfile Directives: EXPOSE](#dockerfile-directives-expose)
    - [Container Volume Management](#container-volume-management)
    - [Docker Network: List and Inspect](#docker-network-list-and-inspect)
    - [Docker Network: Create and Remove](#docker-network-create-and-remove)
    - [Docker Network: Assign to Containers](#docker-network-assign-to-containers)
- [Docker Commands and Structures](#docker-commands-and-structures)
    - [Inspect Container Processes](#inspect-container-processes)
    - [Previous Container Management](#previous-container-management)
    - [Controlling Port Exposure on Containers](#controlling-port-exposure-on-containers)
    - [Naming our Containers](#naming-our-containers)
    - [Docker Events](#docker-events)
    - [Managing and Removing Base Images](#managing-and-removing-base-images)
    - [Saving and Loading Docker Images](#saving-and-loading-docker-images)
    - [Image History](#image-history)
    - [Taking Control of Our Tags](#taking-control-of-our-tags)Pushing to DockerHub
    - [Pushing to DockerHub](#pushing-to-dockerhub)
- [Integration and use Cases](#integration-and-use-cases)
    - [Building a Web Farm for Development and Testing - Prerequisites](#building-a-web-farm-for-development-and-testing-prerequisites)
    - [Building a Web Farm for Development and Testing - Part 1](#building-a-web-farm-for-development-and-testing-part-1)
    - [Building a Web Farm for Development and Testing - Part 2](#building-a-web-farm-for-development-and-testing-part-2)
    - [Building a Web Farm for Development and Testing - Part 3](#building-a-web-farm-for-development-and-testing-part-3)
    - [Building a Web Farm for Development and Testing - Part 4](#building-a-web-farm-for-development-and-testing-part-4)
    - [Integrating Custom Network in Your Docker Containers](#integrating-custom-network-in-your-docker-containers)
    - [Testing Version Compatibility - Using Tomcat and Java - Prerequisites](#testing-versionccompatibility-using-tomcat-and-java-prerequisites)
    - [Testing Version Compatibility - Using Tomcat and Java - Part 1](#testing-version-compatibility-using-tomcat-and-java-part-1)
    - [Testing Version Compatibility - Using Tomcat and Java - Part 2](#testing-version-compatibility-using-tomcat-and-java-part-2)
    - [Testing Version Compatibility - Using Tomcat and Java - Part 3](#testing-version-compatibility-using-tomcat-and-java-part-3)


# Learning the Basics of Docker
---

##Introduction to Docker
[(Back to Top)](#table-of-contents)
Skip

##Containers Vs VirtualMachines
[(Back to Top)](#table-of-contents)
Skip

##Docker Architecture
[(Back to Top)](#table-of-contents)
Skip

##Docker Installation
[(Back to Top)](#table-of-contents)
Skip

##Creating our First Image
[(Back to Top)](#table-of-contents)

See our images
```
docker images
```
Check Version
```
docker version
```

Let's us know API version and google-go version for docker daemon.
```
docker info
```

- Locations
    - **Where Container Details are Located**
        - `var/lib/docker` is where all `images/containers` are stored
        - cd `var/lib/docker/containers/<hex>`
    - **Where Images are Located**
        - ls `/var/lib/docker/image/aufs/imagedb/content/sha256`

Whats Running
```
docker ps
```

What has run (With no name passed, it makes up a name)
```
docker ps -a
```

Refer to a Container by `CONTAINER_ID` or `NAME`

####Pull an image
```
docker pull ubuntu (would be latest)
docker pull ubuntu:trusty (Or any tag listed in dockerhub)
```

"Container Layers build the docker image"

####Launch container
(i = interactive, -t = attach to terminal)
This will launch the container, but not keep it running once we exit
```
docker run -it ubuntu:xenial /bin/bash
```

```
docker ps -a
```

Grab name name of what we ran, eg 'adoring_einstein'

```
docker restart adoring_einstein
docker ps
docker attach adoring_einstein  (Logs us in)
```

####Keep the container running in Background
(disconnected/daemonized)

```
docker run -itd ubuntu:xenial /bin/bash
```

####Run another Instance

```
docker run -itd ubuntu:xenial /bin/bash
```

####Info about the Base image
```
docker inspect ubuntu:xenial
docker ps  (See the different names)
```

####Info about Container
```
docker ps -a
docker inspect compassionate_bhaskara
docker inspect compassionate_bhaskara | grep IP
```

####Going into container

```
docker attach compassionate_bhaskara
<Enter>
```

####Stopping Containers
```
docker stop <name>
docker ps -a
```

####Searching for containers
```
docker search ruby
docker search training/sinatra
```

####Instance Examples
```
docker pull training/sinatra
docker run -it training/sinatra /bin/bash
gem
gem list --local
```

####Create a file in this instance
```
cd /root
echo "Testing" > test.txt
exit
```

####Notice new instances don't have our old items
```
docker run -it training/sinatra /bin/bash
ls /root
```

####We can reboot our old instance
```
docker ps -a
docker restart sad_bohr
docker attach sad_bohr
ls /root         ;(Our test.txt remains)
```

##Packaging a custom container
[(Back to Top)](#table-of-contents)

Instantiate
```
docker ps -a
docker run -it ubuntu:xenial /bin/bash
cd /root
echo "This is version 1 of our custom image" > image_version.txt
apt-get update
apt-get install telnet openssh-server

adduser test

which sshd
which telnet

cat /etc/group | grep test

docker ps -a

docker restart compassionate_cori
docker attach compassionate_cori

docker commit -m "Already installed SSH and created test user" -a "imboyus" compassionate_cori imboyus/ubuntusshd:v1

docker images

docker run -it imboyus/ubuntusshd:v1 /bin/bash
```

####Build a Dockerfile from box

```
vim Dockerfile
```

**Dockerfile**
```
# This is a ubuntu image with SSH already installed
FROM ubuntu:xenial
MAINTAINER imboyus <imboyus@gmail.com>
RUN apt-get update
RUN apt-get install -y telnet openssh-server
```

```
docker build -t="imboyus/ubuntusshdonly:v2" .
docker images
docker run -t imboyus/ubuntusshdonly:v2 /bin/bash
```

##Running Container Commands with Docker
[(Back to Top)](#table-of-contents)

```
docker images
docker run -it ubuntu:xenial /bin/bash
```

```
ps
```
Processes (`ps`) are contained within the docker container, but the perfomance (`top`)
shows the host system performance.

####See the logs from a container (Running or not running)
```
docker ps -a
docker restart cocky_archimedes
docker ps
docker logs cocky_archimedes
```

####Run commands on a running container
Exec only works on running containers nor will it auto-start it.
```
docker exec cocky_archimedes /bin/cat /etc/profile

How to know for sure:
---------------------
docker attach cocky_archimedes
vi /etc/profile     (edit something)

docker restart cocky_archimedes
docker exec cocky_archimedes /bin/cat /etc/profile
```

```
docker run ubuntu:xenial /bin/echo "Hello from this Container"
ps -a  (Notice this creates an instance, though not running)

; the name, eg: admiring_meitner

docker logs admiring_meitner
```

####Containerize

```
docker run -d ubuntu:xenial /bin/bash -c "while true; do echo  HELLO; sleep 1; done"
docker ps
docker logs lonely_bose
docker logs lonely_bose | wc -l
docker stop lonely_bose
```

##Exposing Container with Port Redirects
[(Back to Top)](#table-of-contents)

A port must be exposed to the underlying Host.

```
; We know this uses port 80

docker pull nginx:latest
```

```
docker run -d nginx:latest
docker ps
docker inspect zen_goldberg

Look at the IP Address, eg: 172.17.0.2
```

Install Text Web browser
```
apt-get install elinks
```

See it running
```
elinks http://172.17.0.2
elinks http://localhost (does not find it on underlying host)
```

Run in Daemon mode redirect my local port order to the remote port
```
docker ps
docker stop zen_goldberg

eg: docker run -d -p <local>:<container/remote> nginx:latest

docker run -d -p 8080:80 nginx:latest
docker ps

elinks http://172.17.0.2  (Works)
elinks http://localhost:8080  (Its redirecting traffic from docker daemon)
```

# The Dockerfile, Builds and Network Configuration
---

##Dockerfile Directives: USER and RUN
[(Back to Top)](#table-of-contents)

Dir: `buids/01_RunAsUser`
```
docker pull centos:latest
```

*Dockerfile*

FROM must be the first directive

```
# Dockerfile based on the latest CentOS 7 image - non-privileged user entry
FROM centos:latest
MAINTAINER imboyus@gmail.com

RUN useradd -ms /bin/bash user
USER user
```

docker build -t centos7/nonroot:v1 .
docker run -it centos centos7/nonroot:v1 /bin/bash

_note: When rebuilding an image, it only changes additions and keeps the cache from already existing command builds_

Since a user can't login, we login as user:
```
docker ps -a
docker start cocky_boyd
docker exec -u 0 -it cocky_boyd /bin/bash
```

##Dockerfile Directives: RUN Order of Execution
[(Back to Top)](#table-of-contents)

Dir: `buids/02_CustomMessage`

- Order of Execution
    - `FROM`
    - `MAINTAINER`
    - `RUN`
        - * _If you make a `USER` AFTER this, everything will run as THAT user_

- Important
    - `RUN`: execute at build time, becomes part of the base image.
    - `CMD`: Runs when a container is instantiated, eg: Run an application.

```
docker build -t centos7/config:v1 .
docker run -it centos7/config:v1 /bin/bash
cat /etc/exports.list
```

##Dockerfile Directives: ENV
[(Back to Top)](#table-of-contents)

Dir: `buids/03_JavaInstall`

**When doing updates or installs always do `apt-get update -y` or `yum update -y`**

```
Docker build -t centos7/java8:v1 .      (You'll see red, that's ok)
```

Check the ENV is set for Java, though limited to one user
```
docker images
docker run -it centos7/java8:v1 /bin/bash
env
```

Control System-wide variables
```
Dockerfile:
ENV JAVA_BIN /usr/java/jdk1.8.0/jre
```

```
docker run -it centos7/java8:v2 /bin/bash
env     (Now we see JAVA_HOME global)
```

##Dockerfile Directives: CMD vs RUN
[(Back to Top)](#table-of-contents)

Dir: `buids/04_EchoServer`

- Differences
    - `RUN`: execute at build time, becomes part of the **base image**.
    - `CMD`: Runs when a container is instantiated, or container starts. eg: Run an application. **not part of the build process**

```
docker build -t centos7/echo:v1 .
docker images
docker run centos7/echo:v1  ; Should see an echo from the CMD
```

##Dockerfile Directives: ENTRYPOINT
[(Back to Top)](#table-of-contents)

Dir: `buids/05_Entry`

Sets default application used everytime a container is created even if you ask it to do something else.

```
docker build -t centos7/entry:v1 .
docker images

; Test these
docker run centos7/entry:v1
docker run centos7/echo:v1  /bin/bash echo "See me?"      ; Yes (Notice this is /echo)
docker run centos7/entry:v1  /bin/bash echo "See me?"     ; No
docker run centos7/entry:v1                               ; Outputs default CMD
```

- Different between ENTRYPOINT and CMD
    - `ENTRYPOINT` - Runs no matter what
    - `CMD` - CAN be overwritten for a command, eg: in an echo example above

##Dockerfile Directives: EXPOSE
[(Back to Top)](#table-of-contents)

Dir: `buids/06_ApacheInstallation`

- Commands
    - `-P` is to run/remap any ports in the container

```
docker build -t centos7/apache:v1 .
docker images

# Run container as daemon so it doesn't exit and think it's done

docker run -d --name apacheweb1 centos7/apache:v1
docker ps
docker inspect apacheweb1 | grep IPAdd
```
__Dont worry about the red GPG keys__

See it all works
```
elinks http://172.17.0.3
docker exec apacheweb1 /bin/cat /var/www/html/index.html
docker ps    (Container still running)
```

No Ports Exposed, so:
```
docker stop apacheweb1

docker run -d --name apacheweb2 -P centos7/apache:v1
docker ps    (no ports exposed still)
docker stop apacheweb2
```

Manually Remap ports outside of dockerfile
````
docker run -d --name apacheweb3 -p 8080:80 centos7/apache:v1
docker ps    (ports are exposed)
elinks http://localhost:8080
```

Dockerfile, add:
```
EXPOSE 80
```

Rebuild and check it out
```
docker build -t centos7/apache:v1 .
docker run -d --name apacheweb4 -P centos7/apache:v1
docker ps
```

##Container Volume Management
[(Back to Top)](#table-of-contents)

Dir: `buids/07_MyHostDir`

- Volumes:
    - Mounts
    - File Systems
    - `-v` Flag
        - 1: Create a mount outside containers FS, exposes whatever we put in there to underlying host,
        - 3: Gives ability to provide direct access from host mount to container (eg vagrant, no NFS needed)

```
docker run -it --name voltest1 -v /mydata centos:latest /bin/bash
df -h  (See a /mydata folder which is outside the normal container)

echo "this is a test container file" > /mydata/mytext.txt
exit
```

Where is `/mydata/mytext.txt?
```
docker inspect voltest1 | grep Source
cd /var/lib/docker/volumes/<hash-code-here>/_data
cat mytext.txt

# This will be in the mount on a base image:
echo "this is from me 2" > host.txt
```

Sync <local-path>:<container-path>
```
docker run -it --name voltest2 -v ~/docker-course/builds/MyHostDir:/mydata centos:latest /bin/bash
```

You can use the `VOLUMES` directive in `Dockerfile`, but local files may not be available
for containers, so it may not always be the best for mass distribution.


##Docker Network: List and Inspect
[(Back to Top)](#table-of-contents)

You can see `docker0` which is a bridge default on `172.17.0.1` below, and it will
grab an available IP from docker0:
```
ifconfig    ( look at docker0)
```

Current networks associated with current host:
```
; These are always unique identifiers
docker network ls

; If you need to see the full hash
docker network ls --no-trunc
```

Get Details about a network:
```
docker network inspect bridge

Outputs:

[
    {
        "Name": "bridge",
        "Id": "f49ee9618b80a2423f7ff7725ab335129b59abc441731161b6a0f5db6dd67d80",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Containers": {
            ...
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

##Docker Network: Create and Remove
[(Back to Top)](#table-of-contents)

```
Right:
man docker-network-create (Use this)

Wrong:
man docker network create (Does not work)
```

Create a network with new subnet to use with a bridge for local adapters:

__Most of the time you create subnets for locally routed interal items.__
```
docker network create --subnet 10.1.0.0/24 --gateway 10.1.0.1 mybridge01
docker network ls

ifconfig    (br-<hash> matches the docker network ls ID)
```

Remove a network, be careful to not remove the default 3 networks. You cannot undo it,
and if you do you are better off re-installing docker completely.

```
docker network rm mybridge01
docker network ls
```

Recreate
```
docker network create --subnet 10.1.0.0/24 --gateway 10.1.0.1 mybridge01
docker network inspect mybridge01
docker network rm mybridge01
```

##Docker Network: Assign to Containers
[(Back to Top)](#table-of-contents)

Control how IP's are assigned to our containers.

Class B network with subnet having large range of addresses, and tell the host
only to assign IP's to containers from a subset range for that network.

(About 190 IP's available)

- `--gateway` is how to route to pass the traffic from the underlying host
- `--driver`
    - `bridge` is for a simple setup
    - `overlay` is for a multi-cluster

```
ip/24 = 1-254 (Class C address, last octet)

docker network create --subnet 10.1.0.0/16 --gateway 10.1.0.1 --ip-range=10.1.4.0/24 --driver=bridge --label=host4network bridge04

docker network ls
docker network inspect bridge04
ifconfig
```

Create a Docker Container and use Network
```
docker run -it --name nettest1 --net bridge04 centos:latest /bin/bash

yum update
yum install -y net-tools
ifconfig   (We have 10.1.4.0)
netstat -rn
ping google.com
cat /etc/resolv.conf

exit
```

Static IP's only work on user created networks.
```
docker run -it --name nettest2 --net bridge04 --ip 10.1.4.100 centos:latest /bin/bash
yum update
yum install -y net-tools
ifconfig    (Should show inet 10.1.4.100)
netstat -rn
```

In a new Terminal with the container running we can do:
```
docker inspect nettest2 | grep IPAddr
ping 10.1.4.100
```

Pre-Register DNS with Static IP's, which helps with provisioning and such by using
a static IP.


# Docker Commands and Structures
---

##Inspect Container Processes
[(Back to Top)](#table-of-contents)

A one time view of what's in top:
```
docker top compassionate_cori
```

Get in the box and rather than `attach` this will not close the system when running
```
docker exec -it compassionate_cori /bin/bash
top
```

With two terminals we can monitor in real-time:
```
docker stats compassionate_cori
```

##Previous Container Management
[(Back to Top)](#table-of-contents)

See only the ID's of containers
```
docker ps -a -q
```

See how many images there are
```
docker ps -a -q  | wc -l
```
####Remove a running container
```
docker run zen_goldberg
docker ps
; See we can interact
docker exec zen_goldberg /bin/cat /etc/profile

docker rm -f zen_goldberg
```

####Remove all containers that have run
There is no command to remove all docker containers.  Yet, we can do this:
```
docker rm `sudo docker ps -a -q`
```

####Stop Docker and Remove Container files
```
sudo systemctl stop docker
cd /var/lib/docker/containers
rm -rf <hash>
sudo systemctl restart docker
```

When the hash folder (`config`) is missing for an existing package docker won't run it.


##Controlling Port Exposure on Containers
[(Back to Top)](#table-of-contents)

Run in disconnected mode
```
docker run -itd nginx:latest
docker ps  (80 and 443 are exposed, but not forwarded)

; Get the IP
docker inspect trusting_volhard | grep IP
```

Run with the ports remapped locally
```
docker run -itd -p 8080 nginx:latest
docker ps  (8080 is mapped)
elinks http://172.17.0.2

docker ps
docker stop drunk_carson
```

Expose ports nicely
```
docker run -itd -p 8080:80 nginx:latest
elinks http://localhost:8080
elinks http://172.17.0.2
```

Expose unlimited ports with `-p`
```
docker run -itd -p 8080:80 -p 8443:443 nginx:latest
```

Expose all ports with `-P`
```
docker run -itd -P nginx:latest  (Exposes all ports)
docker inspect big_leavitt | grep IP
docker ps (See what ports were assigned)

elinks http://172.17.0.3
elinks http://localhost:32770
```

Bind only to local port (The IP from inspect wont work in this case)
```
docker run -itd -p 127.0.0.1:8081:80 nginx:latest
docker ps
elinks http://127.0.0.1:8081
```

Bind to UDP only option (TCP Default)
```
docker run -itd -p 127.0.0.1:8081:80/udp nginx:latest
```

##Naming our Containers
[(Back to Top)](#table-of-contents)

Rather than let docker daemon make `container names` we can control them nicer.

- Image names are always lowercase
- Container names are generally lowercase
    - No spacing, or special characters, Allowed are dash(`-`) and underscore(`_`)

```
docker run -itd --name mycontainername ubuntu:xenial /bin/bash
docker ps
```

####Rename a Previously run container
```
docker ps -a
docker rename trusting_volhard myrenamedcontainer
docker rename <hash-id> newnamehere
```

You cannot change the `container id`, nor should you want to.

####Rename running containers
```
docker ps
docker rename lonely_colden superman
docker ps
```

##Docker Events
[(Back to Top)](#table-of-contents)

Have three terminals open.
```
docker run -itd --name c1 ubuntu:xenial /bin/bash
docker run -itd --name c2 ubuntu:xenial /bin/bash
docker run -itd --name c3 ubuntu:xenial /bin/bash
```

####See Events and Listen
In another terminal run
```
docker events --since '1h'
```

In a third terminal run
```
docker events
```

Run this in the first terminal
```
docker exec -it c1 /bin/bash
```

This will show the event shutdown
```
docker attach c1
exit        (See log terminal)
```

####Filters

```
docker events -f   (shorthand)
docker events --filter=(event, container, image, label, type, volume, network, daemon)
```

```
docker events --filter event=attach
docker attach c3    (See log terminal)
```

####Multiple Filters
Note: exiting `-it` mode will cause a die event.
```
docker events --filter event=attach --filter event=die --filter event=stop

docker stop c3
```

##Managing and Removing Base Images
[(Back to Top)](#table-of-contents)

If you have a container which based upon a base image,
it will have a conflict and you should remove the container first.
```
docker images
docker run -it ubuntu:xenial /bin/bash
exit

docker rmi ubuntu:xenial

Error response from daemon: conflict: unable to remove repository reference "ubuntu:xenial" (must force) - container 7607e77c270a is using its referenced image bd3d4369aebc
```

You can force remove a container base image, and restart
the container.
```
docker rmi -f ubuntu:xenial
```

If a container has multiple references with the same Repository, you must use `-f`
```
docker rmi -f centos7/java8
```

##Saving and Loading Docker Images
[(Back to Top)](#table-of-contents)

We can tar a file and save it, and load it later without having to back it up to a repository

```
docker pull centos:latest
docker run -it centos:latest /bin/bash
exit
docker ps -a           (Find the last run container at top)
```

####Save an Image to File
```
docker commit romantic_mestorf centos:mine
docker rm romantic_mestorf
docker images

These will all do the same thing:
---------------------------------
docker save centos:latest > centos.latest.tar
docker save -o centos.latest.tar centos:latest
docker save --output centos.latest.tar centos:latest
```

####More Compression
```
tar tvf centos.latest.tar

; To save disk space:
gzip centos.latest.tar  (200mb to 70mb)
```

####Load the Saved File
Restore the image
```
docker load --input centos.latest.tar.gz
docker images
```

##Image History
[(Back to Top)](#table-of-contents)

Let's us see the details of the build, only available on images and not containers.
```
docker history centos:mine
docker history --no-trunc centos:mine
docker history --quiet nginx
docker history --quiet --no-trunc nginx
```

##Taking Control of Our Tags
[(Back to Top)](#table-of-contents)

Tagging an Image will share the same Image ID.
```
docker tag 4efb2fcdb1ab mine/centos:v1.0
docker tag mine/centos:v1.0 imboyus.server/centos:v1.0.1b
```

##Pushing to DockerHub
[(Back to Top)](#table-of-contents)

- Head over to [DockerHub](http://dockerhub.com)
- Has same naming conventions as Git, eg (jream/name)
- Create a Repository, make it Private
    - Public repositories don't require a login
- Everything will be empty

To get into DockerHub from terminal
```
docker login
docker logout
```

Create an image and push it
```
docker tag ubuntu:xenial jream/myubuntu
docker login
docker push jream/myubuntu
```

```
docker rmi jream/myubuntu
docker pull jream/myubuntu
```

####Shorthand Login
```
docker login -u="user" -p="12345"
docker login --user="user" --password="12345"

; Optional Server besides DockerHub
docker login --user="user" --password="12345" http://optional.server
```

# Integration and use Cases
---

##Building a Web Farm for Development and Testing - Prerequisites
[(Back to Top)](#table-of-contents)

- Working Locally
- Git
- Modem/Network Routing
- `ifconfig` - get the Ethernet or WAN inet addr
- Nginx Proxy Load Balancer on Multiple Nodes
- Host Multiple Domains over Multiple Ports

##Building a Web Farm for Development and Testing - Part 1
[(Back to Top)](#table-of-contents)

Coming

##Building a Web Farm for Development and Testing - Part 2
[(Back to Top)](#table-of-contents)

Coming

##Building a Web Farm for Development and Testing - Part 3
[(Back to Top)](#table-of-contents)

Coming

##Building a Web Farm for Development and Testing - Part 4
[(Back to Top)](#table-of-contents)

Coming

##Integrating Custom Network in Your Docker Containers
[(Back to Top)](#table-of-contents)

Coming

##Testing Version Compatibility - Using Tomcat and Java - Prerequisites
[(Back to Top)](#table-of-contents)

Coming

##Testing Version Compatibility - Using Tomcat and Java - Part 1
[(Back to Top)](#table-of-contents)

Coming

##Testing Version Compatibility - Using Tomcat and Java - Part 2
[(Back to Top)](#table-of-contents)

Coming

##Testing Version Compatibility - Using Tomcat and Java - Part 3
[(Back to Top)](#table-of-contents)

Coming









