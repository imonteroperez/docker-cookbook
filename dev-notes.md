# Dev notes

Some notes (mainly gists) related with examples of 'Docker Cookbook' book.

## Remove all existing containers and volumes (-v option)

```
 docker stop $(docker ps -q)
 docker rm -v $(docker ps -aq)
```

## Running Hello World in Docker

```
 sudo docker run busybox echo hello world
 sudo docker ps -a
 sudo docker images
``` 

Containers are based on images. An image needs to be passed to the docker run command. In the preceding example, you specify an image called busybox. Docker does not have this image locally and pulls it from a public registry. A registry is a catalog of Docker images that the Docker client can communicate with and download images from. Once the image is pulled, Docker starts a container and executes the echo hello world command

## Running an Ubuntu image

```
 sudo docker run -t -i ubuntu:14.04 /bin/bash
```

## Running a simple HTTP server in detached mode

```
 sudo docker run -d -p 1234:1234 python:2.7 python -m SimpleHTTPServer 1234
```

## Create a simple Dockerfile 

```
 imontero@imontero:~/desarrollo/proyectos/docker-cookbook$ sudo docker build -t busybox2 .
 Sending build context to Docker daemon 64.51 kB
 Step 1/2 : FROM busybox
  ---> 59788edf1f3e
 Step 2/2 : ENV foo bar
  ---> Running in 95ebb4dff14c
  ---> 6d296407383d
 Removing intermediate container 95ebb4dff14c
 Successfully built 6d296407383d

 imontero@imontero:~/desarrollo/proyectos/docker-cookbook$ sudo docker images
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
 busybox2            latest              6d296407383d        24 seconds ago      1.15 MB
 <none>              <none>              10f815c01505        46 hours ago        88.3 MB
 python              2.7                 3c43a5d4034a        2 days ago          908 MB
 debian              wheezy              884b80eb370d        2 days ago          88.3 MB
 busybox             latest              59788edf1f3e        2 weeks ago         1.15 MB
 ubuntu              14.04               c32fae490809        6 weeks ago         188 MB
 openjdk             8-jre-alpine        b1bd879ca9b3        9 months ago        82 MB

 imontero@imontero:~/desarrollo/proyectos/docker-cookbook$ sudo docker run busybox2 env | grep foo
 foo=bar

```

## Running a container linked via other container

```
imontero@imontero:~/desarrollo/proyectos/docker-cookbook$ sudo docker pull wordpress:latest
latest: Pulling from library/wordpress
...
imontero@imontero:~/desarrollo/proyectos/docker-cookbook$ sudo docker pull mysql:latest
latest: Pulling from library/mysql
...

imontero@imontero:~/desarrollo/proyectos/docker-cookbook$ sudo docker run --name mysqlwp -e MYSQL_ROOT_PASSWORD=wordpressdocker -d mysql
eaff6c06fa270421d0c0af71f80495433dc53801da82cc738eb2c08d4f262828
imontero@imontero:~/desarrollo/proyectos/docker-cookbook$ sudo docker run --name wordpress --link mysqlwp:mysql -p 80:80 -d wordpress
243819a3eea79460fa6fac921e131e3dd38f14127687d9473068ef6ab7429526
```

You can now run a WordPress container based on the wordpress:latest image. It will be linked to the MySQL container using the --link option, which means that Docker will automatically set up the networking so that the ports exposed by the MySQL container are reachable inside the WordPress container. Both containers should be running in the background, with port 80 of the WordPress container mapped to port 80 of the host.

# Image creation and Sharing

## Keeping changes made to a container by committing to an image

```
imontero@imontero:~/desarrollo/proyectos/docker-cookbook$ sudo docker run -t -i ubuntu:14.04 /bin/bash
root@7b4c8e11cc7b:/# apt-get update
Ign http://archive.ubuntu.com trusty InRelease
Get:1 http://archive.ubuntu.com trusty-updates InRelease [65.9 kB]
Get:2 http://security.ubuntu.com trusty-security InRelease [65.9 kB]       
Get:3 http://archive.ubuntu.com trusty-backports InRelease [65.9 kB]           
Get:4 http://archive.ubuntu.com trusty Release.gpg [933 B]                     
Get:5 http://archive.ubuntu.com trusty-updates/universe Sources [268 kB]
Get:6 http://archive.ubuntu.com trusty-updates/main amd64 Packages [1387 kB]
Get:7 http://security.ubuntu.com trusty-security/universe Sources [104 kB]     
Get:8 http://archive.ubuntu.com trusty-updates/restricted amd64 Packages [21.4 kB]
Get:9 http://archive.ubuntu.com trusty-updates/universe amd64 Packages [634 kB]
Get:10 http://archive.ubuntu.com trusty-updates/multiverse amd64 Packages [16.0 kB]
Get:11 http://archive.ubuntu.com trusty-backports/main amd64 Packages [14.7 kB]
Get:12 http://security.ubuntu.com trusty-security/main amd64 Packages [967 kB]
Get:13 http://archive.ubuntu.com trusty-backports/restricted amd64 Packages [40 B]
Get:14 http://archive.ubuntu.com trusty-backports/universe amd64 Packages [52.5 kB]
Get:15 http://archive.ubuntu.com trusty-backports/multiverse amd64 Packages [1392 B]
Get:16 http://archive.ubuntu.com trusty Release [58.5 kB]                  
Get:17 http://archive.ubuntu.com trusty/universe Sources [7926 kB]
Get:18 http://security.ubuntu.com trusty-security/restricted amd64 Packages [18.1 kB]
Get:19 http://security.ubuntu.com trusty-security/universe amd64 Packages [338 kB]
Get:20 http://security.ubuntu.com trusty-security/multiverse amd64 Packages [4722 B]
Get:21 http://archive.ubuntu.com trusty/main amd64 Packages [1743 kB]
Get:22 http://archive.ubuntu.com trusty/restricted amd64 Packages [16.0 kB]
Get:23 http://archive.ubuntu.com trusty/universe amd64 Packages [7589 kB]
Get:24 http://archive.ubuntu.com trusty/multiverse amd64 Packages [169 kB]
Fetched 21.5 MB in 4s (4416 kB/s)                       
Reading package lists... Done
root@7b4c8e11cc7b:/# exit
exit
imontero@imontero:~/desarrollo/proyectos/docker-cookbook$ sudo docker commit 7b4c8e11cc7b ubuntu:update
sha256:8b7c9f9a43868619a14f047f78deac104fc8a70f9b355ee6579da6650fc19cc6
imontero@imontero:~/desarrollo/proyectos/docker-cookbook$ sudo docker diff 7b4c8e11cc7b
C /root
A /root/.bash_history
C /tmp
C /var
C /var/cache
C /var/cache/apt
D /var/cache/apt/pkgcache.bin
D /var/cache/apt/srcpkgcache.bin
C /var/lib
C /var/lib/apt
C /var/lib/apt/lists
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty-backports_InRelease
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty-backports_main_binary-amd64_Packages.gz
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty-backports_multiverse_binary-amd64_Packages.gz
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty-backports_restricted_binary-amd64_Packages.gz
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty-backports_universe_binary-amd64_Packages.gz
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty-updates_InRelease
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty-updates_main_binary-amd64_Packages.gz
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty-updates_multiverse_binary-amd64_Packages.gz
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty-updates_restricted_binary-amd64_Packages.gz
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty-updates_universe_binary-amd64_Packages.gz
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty-updates_universe_source_Sources.gz
C /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty_Release
C /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty_Release.gpg
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty_main_binary-amd64_Packages.gz
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty_multiverse_binary-amd64_Packages.gz
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty_restricted_binary-amd64_Packages.gz
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty_universe_binary-amd64_Packages.gz
A /var/lib/apt/lists/archive.ubuntu.com_ubuntu_dists_trusty_universe_source_Sources.gz
C /var/lib/apt/lists/lock
C /var/lib/apt/lists/partial
A /var/lib/apt/lists/security.ubuntu.com_ubuntu_dists_trusty-security_InRelease
A /var/lib/apt/lists/security.ubuntu.com_ubuntu_dists_trusty-security_main_binary-amd64_Packages.gz
A /var/lib/apt/lists/security.ubuntu.com_ubuntu_dists_trusty-security_multiverse_binary-amd64_Packages.gz
A /var/lib/apt/lists/security.ubuntu.com_ubuntu_dists_trusty-security_restricted_binary-amd64_Packages.gz
A /var/lib/apt/lists/security.ubuntu.com_ubuntu_dists_trusty-security_universe_binary-amd64_Packages.gz
A /var/lib/apt/lists/security.ubuntu.com_ubuntu_dists_trusty-security_universe_source_Sources.gz
```

