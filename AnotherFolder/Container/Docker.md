# Docker
_Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly._
![](../zz/images/docker_overview.png)

## Docker Basic Commands
Pull Docker image: `docker pull nginx`

Start container detached (without switching the context to the container): `docker run -tid nginx`

Run something in a container (example /bin/bash): `docker exec -ti <CONTAINER_ID> /bin/bash`

Remove Container: `docker rm -f <CONTAINER_ID>`
Remove Container Image: `docker rmi <IMAGE_ID>`

Container Port forwarding (<EXTERNAL_PORT>:<INTERNAL_PORT>): `docker run -tid -p 8080:80 --name test_webserver nginx`

Docker logs (watchdog): `docker logs <CONTAINER_ID> --tail 100 -f`


## Create Docker Container Image
**Checklist for Images**
* Update the image (`apt-get update && apt-get upgrade`)
* Clean update cache (`apt-get autoremove -y && apt-get clean && apt-get autoclean`)
* Add User and Group and container must run as user instead of root (always with User-ID instead of User-Name)
* Port greater then 1024 (ports less then 1024 are reserved for priviliged users)
* Use as less layers (commands) as possible (this decrease build time of an image)


### Dockerfile Example - Base Image (patched)
```dockerfile
FROM ubuntu:21.10  
RUN apt-get update && \
    apt-get upgrade -y && \
	apt-get autoremove -y && \
	apt-get clean && \ 
	apt-get autoclean && \ 
	rm -rf /var/lib/apt/lists/* && \
	addgroup --gid 10000 --system testgroup && \
	adduser --uid 10000 --no-create-home --system --ingroup testgroup testuser  
USER 10000
```

Build: `sudo docker build -t updated_ubuntu:v0.1 .`


### Dockerfile Example - Running a Bash Script and after Die
```dockerfile
FROM ubuntu:21.10  
RUN apt-get update && \
    apt-get upgrade -y && \
	apt-get autoremove -y && \
	apt-get clean && \ 
	apt-get autoclean && \ 
	rm -rf /var/lib/apt/lists/* && \
	addgroup --gid 10000 --system testgroup && \
	adduser --uid 10000 --no-create-home --system --ingroup testgroup testuser  
COPY ./test.bash /opt/test.bash  
RUN chown 10000:10000 /opt/test.bash && chmod u+x /opt/test.bash  
USER 10000  
ENTRYPOINT ["/opt/test.bash"]
```

Build:  `sudo docker build -t updated_ubuntu:v1.0 .`



### Layers
![[../zz/images/docker_layers.png]]
Basically, a layer, or _image layer_ is a change on an image, or an **intermediate image**. Every command you specify (`FROM`, `RUN`, `COPY`, etc.) in your Dockerfile causes the previous image to change, thus creating a new layer. You can think of it as staging changes when you're using git: You add a file's change, then another one, then another one...

> **Important:** Use as less layers (commands) as possible (this decrease build time of a image).

If you create a Docker image and in your build process you want to compile for example some code in Go. Don't use a Go Docker image or install Go in your image.
Although there is nothing technically wrong with the above process, the final image and the resulting container are bloated with layers created while building/preparing the project artefact that are not necessary for the project’s runtime environment.  
Multi-stage builds allow you to separate the creation/preparation phases from the runtime environment. More information in the next chapter...

#### Docker Multi-Stage Builds
![[../zz/images/docker_multistage.png]]
You can still have a single Dockerfile to define your complete build workflow. However, you can copy artefacts from one stage to another while discarding the data in layers you don’t need.

**Example:**
_The ratelimiter binary is built inside the golang based environment and then used in an alpine environment._
```dockerfile
FROM golang:1.10.4 as build
RUN CGO_ENABLED=0 GOOS=linux go get -v github.com/lyft/ratelimit/src/service_cmd

FROM alpine:3.8 AS final
WORKDIR /ratelimit/config
RUN apk --no-cache add ca-certificates
COPY --from=build /go/bin/service_cmd /usr/local/bin/ratelimit
```

*More information's: [How to Build Optimal Docker Images](https://www.metricfire.com/blog/how-to-build-optimal-docker-images/)*




## Troubleshooting
### Exploring Docker Images
1. Figure out what kind of shell is in there `bash` or `sh`...
    Inspect the image first: `docker inspect name-of-container-or-image`
    Look for `entrypoint` or `cmd` in the JSON return.
2. Then do: `docker run --rm -it --entrypoint=/bin/bash name-of-image`
    once inside do: `ls -lsa` or any other shell command like: `cd ..`

*The `-it` stands for interactive... and TTY. The `--rm` stands for remove container after run.*

If there are no common tools like `ls` or `bash` present and you have access to the `Dockerfile` simple add the common tool as a layer. example (alpine Linux):
```
RUN apk add --no-cache bash
```