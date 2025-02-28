+++
date = '2025-02-25T10:56:50+01:00'
draft = false
tags = ['Docker', 'Dockerfile', 'Images', 'Containers']
title = 'Reducing Docker image size'
+++

## Overview

[Docker](https://www.docker.com/) containers have revolutionized application deployment, but oversized images can lead to slower deployments, increased bandwidth costs, and reduced performance. This post provides practical techniques to significantly reduce your [Docker](https://www.docker.com/) images size while maintaining functionality. I'll  also review some of the best tools available for [Docker](https://www.docker.com/) image optimization.

[Docker](https://www.docker.com/), as a container engine, allows you to package code along with its dependencies into a container that can run seamlessly across different environments. It simplifies the process of bundling everything needed to run your application in one place, making deployment faster and easier.

At the heart of [Docker](https://www.docker.com/) image is a simple configuration file called a [Dockerfile](https://docs.docker.com/reference/dockerfile/). This file contains instructions for building the image, including specifying the base image, installing dependencies, and running necessary commands.

## **Why Docker Image Size Matters**  

Optimizing [Docker](https://www.docker.com/) image size isn't just about saving disk space, it directly impacts your container ecosystem's performance, security, and cost efficiency. [Docker](https://www.docker.com/) packages your application code and dependencies into portable containers that run consistently across environments. However, when [Docker](https://www.docker.com/) images become excessively large, they can:  

- **Consume excessive storage resources** – Bloated [Docker](https://www.docker.com/) images place a heavy burden on storage infrastructure. They increase registry storage costs, cause layer duplication, and quickly fill developers' machines—requiring frequent cleanups and maintenance. Larger images also complicate backup processes, increase disk space demands for CI/CD caching, and overwhelm artifact repositories, potentially displacing other critical resources.  

- **Increase network transfer times** – Non-optimized [Docker](https://www.docker.com/) images take longer to push to registries, pull to local machines, and transfer between environments. This increased network load slows down deployments and synchronization across environments, reducing overall efficiency in both CI/CD pipelines and local development workflows.  

- **Performance degradation** - Larger [Docker](https://www.docker.com/) images degrade runtime performance by slowing down container orchestration platforms, such as [Kubernetes](https://kubernetes.io/) or [Docker Swarm](https://docs.docker.com/engine/swarm/), leading to delayed scheduling of containers. This also affects horizontal scaling, making it less responsive during traffic spikes. Noticeable latency can occur during auto-scaling events or disaster recovery scenarios, as the system struggles to manage and deploy the larger images efficiently.

- **Security implications** - Also bloated [Docker](https://www.docker.com/) images weaken security posture by expanding the attack surface with unnecessary packages and dependencies. As more components are added, vulnerability management becomes more complex, requiring additional monitoring efforts. Container security scanning processes demand more resources and time to analyze these larger images, increasing the overall security overhead.

- **Slow down build and deployment pipelines** – Larger [Docker](https://www.docker.com/) images significantly slow CI/CD pipelines by increasing the time required to build and push images to registries. This delays developer feedback loops, making testing and debugging slower due to longer image builds and container startup times. Local testing also becomes cumbersome, as developers experience slower pull times for updated images, reducing productivity and slowing iteration cycles.

## How to reduce Docker image size?

To achieve the right balance between performance, resource consumption, and enhancing security, there are several strategies you can implement that effectively shrink your [Docker](https://www.docker.com/) images while ensuring they remain fully functional. These strategies focus on eliminating unnecessary files, streamlining dependencies and optimizing build processes without compromising the performance or integrity of your application.

In the following sections, we'll explore some of the most effective techniques to achieve a lean, efficient [Docker](https://www.docker.com/) images that meets both performance and operational requirements.

### Use a minimal base image

Probably, one of the most effective ways to reduce [Docker](https://www.docker.com/) image size is by starting with a minimal base image, instead of using heavier or more comprehensive images like [Ubuntu](https://ubuntu.com/) or [Debian](https://www.debian.org/), consider using a lightweight alternative such as [Alpine Linux](https://www.alpinelinux.org/). 

Minimal base images, like [Alpine Linux](https://www.alpinelinux.org/), contain only the essential tools and libraries required to run your application, without the overhead of unnecessary packages. 

By choosing a stripped-down image, you can significantly reduce the size of your [Docker](https://www.docker.com/) image, improve build times, and minimize the attack surface, all while retaining the core functionality needed to run your application. Furthermore, opting for a minimal base image ensures that you only add the dependencies your application truly requires, which leads to cleaner, more efficient images overall.

The [Alpine Linux](https://www.alpinelinux.org/) base image used to illustrate this section, can be as small as **12.1MB**. It’s not just small; it’s very secure as well because has been built with security in mind, featuring a minimal set of packages and tools. Here's an example of how small it is:

```bash
~ docker images                                                                                                                         
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
alpine       latest    a8560b36e8b8   11 days ago   12.1MB
```
On the other hand, [Debian](https://www.debian.org/) and [Ubuntu](https://ubuntu.com/) are larger base images, as they come with more pre-installed software packages. The [Debian](https://www.debian.org/) image, at **181MB**, provides a stable and versatile base with a more extensive set of packages that may be needed for certain applications. It's often favored for its stability, but this comes at the cost of a larger size.

```bash
~ docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
debian       latest    35286826a88d   36 hours ago   181MB
```

[Ubuntu](https://ubuntu.com/), being one of the most widely used Linux distributions, is similarly large, with the latest version weighing in at **117MB**. [Ubuntu](https://ubuntu.com/) offers ease of use and broader support for various tools and applications. While this makes it a good choice for many environments, its size is substantially larger than  [Alpine Linux](https://www.alpinelinux.org/), making it less ideal for applications that prioritize compactness and speed.

```bash
~ docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
ubuntu       latest    72297848456d   4 weeks ago    117MB
```

### Use a distroless base image

Another effective way to reduce [Docker](https://www.docker.com/) image size and enhance security is by using distroless images. Unlike traditional base images like [Alpine Linux](https://www.alpinelinux.org/), [Debian](https://www.debian.org/) or [Ubuntu](https://ubuntu.com/), distroless images contain only the application and its runtime dependencies, leaving out package managers, shells, and unnecessary utilities. 

This results in a much smaller image footprint and reduces the attack surface by eliminating extraneous components. 

Distroless images are ideal for running applications in production environments where you want to minimize vulnerabilities and ensure that only the necessary runtime components are included.

For example, a distroless image for a Python application will only contain the Python runtime and the application itself, without additional tools like bash or apt-get, making it more secure and efficient.

Here's an example of the distroless image for Python:

```bash
~ docker images
REPOSITORY                  TAG       IMAGE ID       CREATED   SIZE
gcr.io/distroless/python3   latest    981541f9c36f   N/A       83.3MB
```

In comparison to standard base images, distroless images can be significantly smaller, as they contain only the essential elements required for your application to run, without the additional bulk of a general-purpose operating system.

### Leverage Multi-stage builds

One of the most effective ways to reduce the size of your [Docker](https://www.docker.com/) images is by using multi-stage builds. 

Multi-stage builds allow you to separate the build process from the final image, ensuring that only the necessary artifacts are included in the final container, while discarding any non-essential files, dependencies, or build tools. This practice is invaluable for optimizing image size, improving security, and reducing the surface area for potential vulnerabilities.

The following _Dockerfile_ uses a multi-stage build approach to create a more efficient and secure image for deploying a Python application.

```dockerfile

# Stage 1: Build
FROM python:3.14-slim AS build

# Set the working directory inside the container
WORKDIR /app

# Copy the requirements file and install the dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code
COPY . .

# Perform any additional build tasks if necessary
# For example, compiling assets or running tests

# Stage 2: Final Image
FROM python:3.14-slim AS final

# Set the working directory inside the container for the final image
WORKDIR /app

# Copy the installed dependencies from the build stage
COPY --from=build /usr/local/lib/python3.9/site-packages /usr/local/lib/python3.9/site-packages

# Copy the application code from the build stage
COPY --from=build /app .

# Expose port 80
EXPOSE 80

# Define the environment variable
ENV NAME World

# Command to run the application
CMD ["python", "app.py"]
```

In the build stage, the _Dockerfile_ starts with a minimal Python 3.14 base image (python:3.14-slim).

The ```WORKDIR``` directive sets _/app_ as the working directory inside the container, where all subsequent commands will be executed.

The _requirements.txt_ file is then copied into the container, and Python dependencies are installed with pip. Using Docker withe the ```--no-cache-dir``` option ensures that unnecessary package installation caches are not included, further reducing image size.

After installing the dependencies, the rest of the application code is copied into the container. This stage is focused on building the environment and dependencies needed for the application.

In the final stage, the _Dockerfile_ again starts with the same minimal Python 3.14 base image. The ```WORKDIR``` is set to _/app_, and the dependencies installed during the build stage are copied over using the ```COPY --from=``` build directive, which pulls files from the previous build stage.

Only the application code and installed dependencies are included in this final image, keeping it lean and free from any build tools or unnecessary files.

## Docker layer optimization

[Docker](https://www.docker.com/) images are built using a layered architecture, where each layer represents a change made by a _Dockerfile_ instruction. Every time you execute commands like ```RUN```, ```COPY``` or ```ADD``` in your _Dockerfile_, [Docker](https://www.docker.com/) creates a new layer that records only the changes from the previous layer. This is an important concept because each layer not only consumes storage but also adds time to the image build process.

Each layer is immutable, meaning once it’s created, it cannot be changed, only new layers can be added on top,  allowing [Docker](https://www.docker.com/)  to efficiently manage images by reusing unchanged layers across multiple builds, which improves both build speed and storage efficiency.

Additionally, layers are cached and reused, meaning that unnecessary layers can increase the overall size and inefficiency of the image.

### Combining related RUN commands

In the following example,each ```RUN``` command create a layer in the image making the image larger because of the additional layers:

```dockerfile

# Use an official Python runtime as the base image
FROM python:3.14-slim

# Update the system
RUN apt-get update

# Install Python3
RUN apt-get install -y python3

# Install pip
RUN apt-get install -y python3-pip

# Execute apt-get clean 
Clean the cache to reduce image size
RUN apt-get clean 

# Clean the cache to reduce image size
RUN rm -rf /var/lib/apt/lists/*

# Set the working directory
WORKDIR /app

# Copy the application code into the container
COPY . /app

# Install Python dependencies from the requirements.txt file
RUN pip3 install -r requirements.txt

# Expose a port (if your application needs it)
EXPOSE 8000

# Default command to run when the container starts
CMD ["python3", "app.py"]
```

To optimize the size of your [Docker](https://www.docker.com/) image, it’s a best practice to concatenate multiple ```RUN``` commands into a single statement making the _Dockerfile_ more efficient but also improving readability.


```dockerfile
# Use an official Python runtime as the base image
FROM python:3.14-slim

# Update the system, install Python3, pip, clean cache, 
# and remove unnecessary files

RUN apt-get update && apt-get install -y python3 python3-pip && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Set the working directory
WORKDIR /app

# Copy the application code into the container
COPY . /app

# Install Python dependencies from the requirements.txt file
RUN pip3 install -r requirements.txt

# Expose a port (if your application needs it)
EXPOSE 8000

# Default command to run when the container starts
CMD ["python3", "app.py"]
```

This approach has several benefits:

- **Smaller image ize** - Temporary files and cache data created during intermediate steps are removed before a new layer is committed.
- **Faster build time** - Docker caches layers, and fewer layers mean fewer steps for Docker to process.
- **Improved efficiency** - Reduces duplication and ensures that cleanup happens in the same layer, keeping the image lean.

### Optimizing the use of COPY and ADD instructions

The ```COPY``` and ```ADD``` instructions are used to transfer files from your local filesystem to the [Docker](https://www.docker.com/) image. Each use of these commands creates a new layer, so optimizing how you transfer files can significantly reduce image size and improve efficiency.

While ```COPY``` and ```ADD``` share similar functionality in [Docker](https://www.docker.com/), as both copy files or directories from the local system into a [Docker](https://www.docker.com/) image, there are key differences between them.

The ```COPY``` command is straightforward—it simply transfers files or directories from your local environment to the [Docker](https://www.docker.com/) image without any additional features. 

Here's a basic _Dockerfile_ using the ```COPY``` instruction:

```dockerfile
# Use a minimal base image
FROM alpine:latest

# Set the working directory inside the container
WORKDIR /app

# Copy local files to the container
COPY . /app

# Run a command (example: starting a script)
CMD ["sh", "start.sh"]

```

In contrast, ```ADD``` offers extra capabilities: it can automatically extract compressed files (e.g., .tar.gz archives) and fetch files from remote URLs. These additional features can be useful but may also introduce complexity and unintended behavior if not carefully managed.

According to [Docker's official documentation](https://docs.docker.com/build/building/best-practices/#add-or-copy), it is recommended to use ```COPY``` instead of ```ADD``` due to its simplicity and transparency. The ```ADD``` command should only be used when you specifically need to extract a local compressed file.

Here's a basic _Dockerfile_ using the ```ADD``` instruction:

```dockerfile
# Use a base image
FROM alpine:latest

# Set the working directory inside the container
WORKDIR /app

# Add a local archive and extract it automatically
ADD files.tar.gz /app/

# Run a command (example: listing directory contents)
CMD ["ls", "-l", "/app"]
```

For downloading remote files, combining the ```RUN``` instruction with tools like _wget_ or _curl_ is considered safer and more efficient and  prevents the creation of unnecessary image layers and helps reduce the final image size.

The following example demonstrates how to use the ```RUN``` instruction to download a compressed package from a URL, extract its contents, and remove the archive to keep the image clean:

```dockerfile
# Use a base image
FROM alpine:latest

# Set the working directory inside the container
WORKDIR /app

# Install wget, download, extract the archive, and clean up in a single layer
RUN apk update && apk add --no-cache wget tar && \
    wget -O /tmp/files.tar.gz https://example.com/files.tar.gz && \
    tar -xzf /tmp/files.tar.gz -C /app/ && \
    rm -rf /tmp/files.tar.gz

# Run a command (example: listing directory contents)
CMD ["ls", "-l", "/app"]
```

