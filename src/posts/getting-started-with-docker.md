---
title: Getting Started with Docker
metaDesc: A quick look at what Docker is and how to get started.
date: '2017-05-31'
tags: 
- docker
- dotnet
---

It's more than likely that you will have already heard about Docker, but what exactly is it and what does it offer you? Well, Docker is a container platform. But, what is a container? Well, a container is an isolated image that includes everything you need to run your app. So, that includes your code, the runtime, any libraries and settings. Unlike a Virtual Machine (VM), you do not need to bundle a Guest OS. This keeps things lightweight and you can run multiple containers on a single host. The premise for this, is that containerised software will always run the same, regardless of the environment so you shouldn't get the fabled 'works on my machine' issue. 

### Downloading Docker

Docker is available in two editions: Community Edition (CE) and Enterprise Edition (EE). Docker CE is available for Mac, Windows and Linux distros. I'm going to be talking about the Windows version in this article, so grab the stable version of Docker for Windows from here: 

[https://store.docker.com/editions/community/docker-ce-desktop-windows](https://store.docker.com/editions/community/docker-ce-desktop-windows)

Docker uses Windows-native Hyper-V virtualisation, so if you haven't enabled this feature it will prompt you to enable this and restart. Once that is done, you'll notice that Docker is running in your taskbar. A quick click on 'About Docker' and you should see confirmation of the current version.

### Running Docker

To interact with Docker, you can use the `docker` commands in a terminal such as Powershell. Here's how to run the simple Hello World example.

```
PS C:\Users\tmorris> docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
78445dd45222: Pull complete
Digest: sha256:c5515758d4c5e1e838e9cd307f6c6a0d620b5e07e6f927b07d05f6d12a1ac8d7
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://cloud.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

Some useful commands:

- `docker version`: see version information for Docker
- `docker ps`: display running containers
- `docker rm <container>`: remove a container
- `docker images`: display images
- `docker rmi <image>`: remove an image

To enable tab completion within Powershell, download [posh-docker](https://github.com/samneirinck/posh-docker).

### An example

Here is a slightly more involved example that shows a [simple application](https://github.com/dotnet/dotnet-docker-samples/blob/master/dotnetapp-prod/Program.cs) running .NET Core in a Linux container. All it does is display a brief welcome by [dotnetbot](https://github.com/dotnet-bot).

`docker run microsoft/dotnet-samples`

![Docker dotnetbot](/images/docker_dotnetbot.png)

### Options

Docker opens up a few possibilities in the developer workflow. Here's some usage cases:

- Create a consistent development environment between your team
- Pull in your dependencies as neatly packaged Docker images
- Isolate concerns, develop and deploy independently with a microservices architecture
- Build and test your apps with Bitbucket Pipelines (or similar) to provide Continuous Integration (CI)
- Scale up and manage your infrastructure with Docker Swarm

### Summary

Docker is gaining momentum amongst the dev community. I've given a quick overview here about how to get started and how you can incorporate Docker into your workflow. Give it a try :)

#### References

- [Docker: What is Docker](https://www.docker.com/what-docker)
- [Docker: Getting started](https://docs.docker.com/docker-for-windows/)
- [Docker: Overview](https://docs.docker.com/engine/docker-overview/)
- [Microsoft: Using .NET and Docker together](https://blogs.msdn.microsoft.com/dotnet/2017/05/25/using-net-and-docker-together/)
- [Bitbucket Pipelines](https://bitbucket.org/product/features/pipelines)