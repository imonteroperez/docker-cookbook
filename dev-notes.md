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
