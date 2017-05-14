# dotnet-mono-dockerclient

The dotnet-mono-dockerclient docker image contains:
1. .NET Core base image
2. Mono
3. A docker client

This make it a great image to run automated build of .NET Core projects using [Cake](http://cakebuild.net/) (thus that depends on mono) and that generates docker images. 

Since this image contains a docker client, when you start a container based on `dotnet-mono-dockerclient`, it will be possible to run `docker` commands from within your container. This is what's called by some [docker in docker](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/). To be able to do this, you will need to start the container with the good options so that it can connect to an docker server running elsewhere. Two options possible are:

1. Use the `-H` option of `docker run`
1. Use the `-v /var/run/docker.sock:/var/run/docker.sock` option of `docker run` as described [here](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/). This is my prefered option since it does not require the docker server running on your build machine to receive connections from outside of the server, thus keeping your server secure.

Here is a typical example of how to use this capability:

1. Start a container that contains your build script (`your_build.sh`). Note the two -v options. This assumes that you have a script named `your_build.sh` that exists at the root of your git repository.
````
docker run -it --rm \
           -v /var/run/docker.sock:/var/run/docker.sock \
           -v $(pwd):/git \
           -w /git \
           mabead/dotnet-mono-dockerclient:PROPER_TAG  \
           your_build.sh 
```` 
2. From within `your_build.sh`, you can then execute other docker commands that will be executed on the docker server running on your host. For example, you can build new containers from within your build container!
````
docker build
````
