# Docker

Container -> seperate operating system will all necessary packages 

- containenerized Application 
  - only app and dependencies we need less system resource that virutal machine

https://docs.docker.com/engine/install/ubuntu/

docker Compose as well

```bash
docker run hello-world 
docker ps # show active container 
```

docker break down after finishes its job - but it is not destroyed 

```bash
docker run --rm hello-world
docker run --rm ubuntu:18.04 # pull specific 
```

## 2. Create Simple Docker File 



```json
{
    "posts" : [
        {
            "title": "Post",
            "author": "Piotrek"
        }
    ],
    "profile": {
        "first_name" : "Piotrek",
        "last_name": "Sad",
        "email": "piotrek@sad.com"
    }
}
```



```bash
mkidr folder
touch Dockerfile # to provide instruction for our container 
touch db.json 
docker build.
docker image list #provide list of images available on our machine
docker run -rm <ID>
docker ps # list of docker that currently run 
docker stop <container id>
docker build . --no-cache #diregard any cache file 
docker image list
docker run --rm -p 3000:3000 <id>
```

 

=> 

```bash
FROM node:latest
WORKDIR /home/server
RUN npm install -g json-server
COPY db.json /home/server/db.json
EXPOSE 3000 # port 
ENTRYPOINT ["json-server", "db.json", "--host", "0.0.0.0"] # main command  "0.0.0.0" - port bridge docker and machine
```

option :

```bash
COPY alt.json /home/server/alt.json
ENTRYPOINT ["json-server", "--host", "0.0.0.0"]
CMD ["db.json"]
```

```bash
docker run --rm -p 3000:3000 json-server alt.json
```

### 3. Setting up Ngnix

container that will run web server 

alpine -linux light weight distro

```bash
FROM ngnix:stable-alpine
ENV NGINXUSER=laravel
ENV NGINXGROUP=laravel
RUN mkdir -p /var/www/html/public

# add attributes
ADD ngnix/default.conf /etc/ngnix/conf.d/default.conf
#sed - editor
# replace user with laravel
RUN sed -i "s/user www-data/user ${NGINXUSER}/g" /etc/ngnix/nginx.conf
# add user to linux shell
RUN adduser -g ${NGINXGROUP} -s /bin/sh -D ${NGINXUSER}
```

```bash
mkdir nginx
touch nginx/default.conf
# pass boilerplace
docker build --no-cache -t laravel-nginx . 
docker run -rm -p 80:80 laravel-nginx
# error - we need source 
# Volume = Virtual mount of filesystem that connect files 
mkdir src
touch src/index.html
docker run -rm -p 80:80 -v <path to project directory/src:/var/www/html/public> laravel-nginx
```

## 4. Docker Compose & MYSQL

tool to define and manage multi-container application 

- `yaml` file 

```bash
touch docker-compose.yml #file below
docker-composer up
mkdir mysql
docker-composer up
```

yml file 

```bash
version: '3.8'
services: 
	nginx:
		build:
			context: .
			dockerfile: nginx.dockerfile # rename our file
		ports:
        	- 80:80
        volumes:
        	- ./src:/var/www/html
	mysql:
		image:mariadb:10.5 #mysql:5.7
		ports:
			- 3360:3306
		environment:
        	MYSQL_DATABASE: laravel
        	MYSQL_USER: laravel
        	MYSQL_PASSWORD: secret
        	MYSQL_ROOT_PASSWORD: secret
        volumes:
        	- ./mysql:var/lib/mysql
        	
```

## 5. PHP container

add another serivce in docker-composer file 

```bash
php: 
	build:
		context: .
		dockerfile: php.dockerfile
	volumes:
		- ./src:/var/www/html
```

php.dockerfile

```bash
FROM php:8-fpm-alpine 
ENV PHPGROUP=laravel
ENV PHPUSER=laravel
RUN adduser -g ${PHPGROUP} -s /bin/sh -D ${PHPUSER}
RUN sed -i "s/user = www-data/user = ${PHPUSER}/g" /usr/local/etc/php-fpm.d/www.conf
RUN sed -i "s/group = www-data/group = ${PHPGROUP}/g" /usr/local/etc/php-fpm.d/www.conf

RUN mkdir -p /var/www/html/public
RUN docker-php-ext-install pdo pdo_mysql
# to run as root: 
CMD ["php-fpm". "-y", "/usr/local/etc/php-fpm.d/www.conf", "-R"] 
```

## Containers

"few features of linux kernel - duck-taped together"



**bare metal**  - physical presence with all bills  / disk swapping / drivers compatibilites

**virtual machines** - with adding new service - new VM on one of our servers

VM - guest os - if crash - crash themsevles 

**public cloud** - public cloud provider like Microsoft Azure or Amazon Web Services. 

- pre-allocated amount of memory and computing power - virtaul cores (vCorse)



**containers**

-  give us many of the security and resource-management features of VMs 
-  we do not need to run a whole other operating system. 
-  usings chroot, namespace, and cgroup to separate a group of  processes from each other. If this sounds a little flimsy to you and  you're still worried about security and resource-management, you're not  alone. But I assure you a lot of very smart people have worked out the  kinks and containers are the future of deploying code.

So now that we've been through why we need containers, let's go through the three things that make containers a reality.

### Chroot

`docker run -it --name docker-host --rm --privileged ubuntu:bionic`

```bash
cat /etc/issue # to check in what distribution we are
mkdir my-new-root
mkdir my-new-root/bin
cp bin/bash my-new-root/bin/
# we need library to work with 
ldd bin/bash # show all libraries we are dependent on
mkdir my-new-root/lib{,64} # crreate 2 dirs
cp /lib/x86_64-linux-gnu/libtinfo.so.5 /lib/x86_64-linux-gnu/libdl.so.2 /lib/x86_64-linux-gnu/libc.so.6 my-new-root/lib
cp /lib64/ld-linux-x86-64.so.2 my-new-root/lib64/
chroot my-new-root/ bash
exit # get out of ch-root env
cp /bin/ls my-new-root/bin 
ld /bin/ls # make sure what should be inside 

cp /lib/x86_64-linux-gnu/libselinux.so.1 /lib/x86_64-linux-gnu/libpthread.so.0 my-new-root/lib
cp /lib/x86_64-linux-gnu/libpcre.so.3 my-new-root/lib


```



### Namepsaces 

```bash
ps # we see 2 process here -> each can see so we can kill process !:)
tail -f secret.txt # start long running process 
# open the same cotaniner in different shell :

# update list of packages: 
 apt-get updated
apt-get install debootstrap -y
# with that tool -> we can set up better root - we will provide minimal installation 
debootstrap --variant=minbase bionic /better-root	
# it is debian bootstrap	
cd better-root
chroot . bash
exit # go back to normal

# unshare->  unshare creates a new isolated namespace from its parent
unshare --mount --uts --ipc --net --pid --fork --user --map-root-user chroot /better-root bash # this also chroot's for u
```

This will create a new environment that's isolated on the system with its own PIDs, mounts (like storage and volumes), and network stack. Now we can't see any of the processes!

```bash
mount -t proc none /proc # process namespace
mount -t sysfs none /sys # filesystem
mount -t tmpfs none /tmp # filesyste
```

Control groups -> isolated environment only gets so much CPU, so much memory, etc. and once it's out of those it's out-of-luck,

