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

