# Qick guide to Docker.

## Intro
Docker is a powerful tool for creating containers (lightweight virtual machines, that use kernel of host OS as their kernel). 

## Basics

Docker image - is a ...
Docker container is an instance of Docker image.

### Creating an image

To create a Docker container you should either:

- pull it from [Docker Hub](https://hub.docker.com/).
- or create it using `docker build` command.

To see all images available locally you should issue `docker images` command.

### Pulling an image

Use `docker pull [image_name]:[tag]` command to fetch an image from Docker Hub.

Tag is used for certain version of an image. By default it's `latest`.

### Running a container

To run a Docker container you should use `docker run [instance]` command. It has a lot of flags. The most useful are:

- `-d` will run container in the background.
- `--name [name]` will create the container with certain name.
- `-p [host_port]:[container_port]` will expose container's ports to host ports. 
  
  For example: \
  `docker run -d -p 8080:8080 jenkins` 

- `-e [VARNAME]=[variable_value]` will set environmental variable inside the container. 
  
  For example:\
  `docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=root -p 3306:3306 mysql:5.7`

- `-v [your_data_directory]:[directory_inside_container]` will mount a `volume` for directory in your container.

  It means that any data in this directory will be kept across container restarts (without volume, it won't).
  For example:\
  `docker run -v your_data_directory:/var/data/elasticsearch elasticsearch`.

  You also may use volumes to provide configuration files like this (`:ro` means that access to volume is read-only): \
  `docker run -d -p 80:80 -v /some/nginx.conf:/etc/nginx/nginx.conf:ro nginx`

## Operating containers

To see the list of running containers you may issue `docker ps` command. \
To see all containers available on the machine you should issue `docker ps -a`.

All following commands accept container ID (or even it's prefix, for your convenience):

- `docker start [id]` - run existing container.
- `docker pause [id]` - pause all processes inside the container.
- `docker stop [id]` - stop a running container.

To remove image or container you may use:

- `docker rm [id]` - remove container.
- `docker rmi [image_id]` - remove image.



