## Learning how Docker works
*These notes are copy-pasted from, or based on, learnning from https://medium.freecodecamp.org/a-beginner-friendly-introduction-to-containers-vms-and-docker-79a9e3e119b, https://docs.docker.com/get-started/, https://tuhrig.de/difference-between-save-and-export-in-docker/*
**Container vs. VM** = A *container* shares the kernel of the host machine with other containers. It takes no more memory than any other executable, making it lightweight. A *VM* runs a full-blown “guest” operating system, sometimes more resources than a developer needs.
**Docker engine** = foundation of docker
**Docker client** = UI
**Dockerfile** = Where you write the instructions to build a Docker image. Once you’ve got your Dockerfile set up, you can use the docker build command to build an image from it.
**Docker image** = Images are *read-only* templates that you build from a set of instructions written in your Dockerfile. Images define both what you want your packaged application and its dependencies to look like *and* what processes to run when it’s launched.
**Container** = A running instance of an image
**Volumes** = Volumes are the “data” part of a container, initialized when a container is created. Volumes allow you to persist+share a container’s data. Data volumes exist as normal directories and files on the host filesystem. So, even if you destroy, update, or rebuild your container, the data volumes will remain untouched. When you want to update a volume, you make changes to it directly.
**Docker container** = Wraps an application’s software into an invisible box with everything the application needs to run: operating system, application code, runtime, system tools, system libraries, and etc. Docker containers are built off Docker images. Since images are read-only, Docker adds a read-write file system over the read-only file system of the image to create a container.
**Save vs. export** = A running instance of an image is called container. You can make changes to a container (e.g. delete a file), but these changes will not affect the image. However, you can create a new image from a running container (and all it changes) using docker commit <container-id> <image-name>.
Export is slightly smaller than save because it is flattened, which means it lost its history and meta-data.
*Export* is used to persist a container (not an image). So we need the container id which we can see like this:
`sudo docker ps -a`
To export a container we simply do:
`sudo docker export <CONTAINER ID> > /home/export.tar`
*Save* is used to persist an image (not a container). So we need the image name which we can see like this:
`sudo docker images`
To save an image we simply do:
`sudo docker save busybox-1 > /home/save.tar`
