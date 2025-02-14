+++
date = '2025-02-12T08:51:34+01:00'
draft = false
title = 'Static Analys of Dockerfiles'
+++

## Overview

[Hadolint](https://github.com/hadolint/hadolint) is an open-source tool designed for static analysis of **Dockerfiles**. Its purpose is to help you identify common errors, security issues, and improve **Docker** image building practices. It works similarly to a "linter," meaning it analyzes your **Dockerfile** code and provides recommendations on how to optimize it and ensure best practices are followed, such as:

- Reducing image size.
- Enhancing security.
- Optimizing image layers.
- Verifying unnecessary commands.

Being based on [ShellCheck](https://www.shellcheck.net/), it also reviews the **Bash** code within the ```RUN``` instructions of your **Dockerfile**, helping to detect potential errors or discouraged practices in shell scripts.

### Installation

If you don't have [Hadolint](https://github.com/hadolint/hadolint) installed yet, you can install on macOS with [Homebrew](https://brew.sh/):

```bash
brew install hadolint
```

### Writin a Dockerfile

To better understand how [Hadolint](https://github.com/hadolint/hadolint) works, let's create a **Dockerfile** that contains several common issues and anti-patterns. This example deliberately includes problems that [Hadolint](https://github.com/hadolint/hadolint) will detect:

```dockerfile
# Using a non-specific base image.
FROM python

# Maintainer information.
MAINTAINER testuser@test.com

# Running as root without necessity.
USER root

# Update package indices.Installing dependencies without cleaning the cache.
RUN apt-get update

# Installing dependencies without cleaning the cache.
Run apt-get install -y curl

# No working directory specified.
WORKDIR /

# Copying files without considering the build context.
COPY . .

# Installing global dependencies unnecessarily.
RUN pip3 install -r requirements.txt

# Exposing an unnecessary port
EXPOSE 8000

# Default command to run when the container starts
CMD python3 app.py
```

This Dockerfile contains several issues that Hadolint will flag:

- Use of deprecated MAINTAINER instruction
- Non-specific base image tag
- Separate RUN commands for apt-get operations
- Missing cleanup of apt cache
- No version pinning for packages
- Potentially unsafe use of root user
- Inefficient copying of build context

When we run [Hadolint](https://github.com/hadolint/hadolint) on this **Dockerfile**, it will help us identify these issues and provide suggestions for improvement. Let's see what happens when we analyze it in the next section.

### Linting a Dockerfile with Hadolint

Let's take a closer look at our problematic **Dockerfile** by running an analysis with [Hadolint](https://github.com/hadolint/hadolint) to identify potential issues and areas for improvement. [Hadolint](https://github.com/hadolint/hadolint) will help us spot common mistakes, security risks, and optimization opportunities in the **Dockerfile**. By running this analysis, we can ensure that the **Dockerfile** adheres to best practices, reduces image size, and improves both security and maintainability. 

Let's run Hadolint:

```bash
hadolint Dockerfile
```

This will run the static analysis and display a list of warnings or issues directly in your terminal.


```bash
Dockerfile:2 DL3006 warning: Always tag the version of an image explicitly
Dockerfile:5 DL4000 error: MAINTAINER is deprecated
Dockerfile:8 DL3002 warning: Last USER should not be root
Dockerfile:11 DL3009 info: Delete the apt-get lists after installing something
Dockerfile:15 DL3059 info: Multiple consecutive `RUN` instructions. Consider consolidation.
Dockerfile:15 DL3008 warning: Pin versions in apt get install. Instead of `apt-get install <package>` use `apt-get install <package>=<version>`
Dockerfile:15 DL3015 info: Avoid additional packages by specifying `--no-install-recommends`
Dockerfile:24 DL3042 warning: Avoid use of cache directory with pip. Use `pip install --no-cache-dir <package>`
Dockerfile:30 DL3025 warning: Use arguments JSON notation for CMD and ENTRYPOINT arguments
```
