# dotnet-mono-dockerclient

The dotnet-mono-dockerclient docker image contains:
1. .NET Core base image
2. Mono
3. A docker client

This make it a great image to run automated build of .NET Core projects using [Cake](http://cakebuild.net/) (thus that depends on mono) and that generate docker images. 

Since this image contains a docker client, when you start a container based on `dotnet-mono-dockerclient`, it will be possible to run `docker` commands from within your container. This is what's called by some [docker-in-docker](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/). With the proper `docker run` options, the docker client that it contains will be able to connect to a docker server running outside of the container. Two options are possible to specify the docker server to use:

1. Set the `DOCKER_HOST` environment variable when calling `docker run`: `docker run -e "DOCKER_HOST=NAME_OF_HOST_RUNNING_DOCKER_SERVER"`
1. Use the `-v /var/run/docker.sock:/var/run/docker.sock` option of `docker run` as described [here](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/). This is my prefered option since it does not require the docker server running on your build machine to accept connections from outside of the server, thus keeping your server secure.

Here is a typical example of how to use this capability:

1. Start a container where you mount your build script (`your_build.sh`). Note the two -v options. This assumes that you have a script named `your_build.sh` that exists at the root of your git repository.
````
docker run -i --rm \
           -v /var/run/docker.sock:/var/run/docker.sock \
           -v $(pwd):/git \
           -w /git \
           mabead/dotnet-mono-dockerclient:PROPER_TAG  \
           ./your_build.sh 
```` 
2. From within `your_build.sh`, you can then execute other docker commands that will be executed on the docker server running on your host. For example, you can build new containers from within your build container!
````
docker build
````
