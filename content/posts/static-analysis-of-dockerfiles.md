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

- Use of deprecated MAINTAINER instruction.
- Non-specific base image tag.
- Separate RUN commands for apt-get operations.
- Missing cleanup of apt cache.
- No version pinning for packages.
- Potentially unsafe use of root user.
- Inefficient copying of build context.

When we run [Hadolint](https://github.com/hadolint/hadolint) on this **Dockerfile**, it will help us identify these issues and provide suggestions for improvement. Let's see what happens when we analyze it in the next section.

### Configuring Hadolint

[Hadolint](https://github.com/hadolint/hadolint) is a highly configurable tool, allowing you to customize its behavior according to your specific needs and requirements. You can create a ```.hadolint.yaml``` configuration file within your project to fine-tune various aspects of its functionality. This file provides several options to optimize how [Hadolint](https://github.com/hadolint/hadolint) works with your **Dockerfiles**, including:

- **Ignored rules:** Specify which linting rules to ignore, giving you control over the rules that matter most for your project.
- **Trusted registries:** Define trusted Docker registries to prevent warnings about certain images, ensuring you’re working with reliable sources.
- **Label schema requirements:** Enforce specific labeling conventions and schema to ensure consistency and compliance with your project’s standards.
- **Severity overrides:** Adjust the severity of certain rules (e.g., set a rule to a "warning" instead of "error") to prioritize issues according to your project's requirements.

Here's an example of a ```.hadolint.yaml``` file that shows you how to configure [Hadolint](https://github.com/hadolint/hadolint) for your project:

```bash
# .hadolint.yaml

# Define ignored rules
ignored:
  - DL3008  # Ignore warning about using apt-get instead of apt
  - DL4006  # Ignore warning about missing labels

# Define trusted registries
trusted-registries:
  - docker.io
  - my-private-registry.com

# Enforce label schema requirements
label-schema:
  description: "A custom Docker image"
  version: "1.0"
  vendor: "Your Company"
  
# Severity overrides
severity:
  DL3003: warning  # Set DL3003 (no labels) to warning instead of error
  DL3026: error    # Keep DL3026 (missing USER) as an error
```

For detailed instructions on configuring these settings and leveraging [Hadolint](https://github.com/hadolint/hadolint) capabilities, refer to the official [Hadolint documentation](https://github.com/hadolint/hadolint?tab=readme-ov-file#rules).

### Linting a Dockerfile with Hadolint

Let's take a closer look at our problematic **Dockerfile** by running an analysis with [Hadolint](https://github.com/hadolint/hadolint) to identify potential issues and areas for improvement. [Hadolint](https://github.com/hadolint/hadolint) will help us spot common mistakes, security risks, and optimization opportunities in the **Dockerfile**. By running this analysis, we can ensure that the **Dockerfile** adheres to best practices, reduces image size, and improves both security and maintainability. 

Let's run [Hadolint](https://github.com/hadolint/hadolint):

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

### Fixing the issues

After analyzing the warnings and errors reported by [Hadolint](https://github.com/hadolint/hadolint), we can create an improved version of our **Dockerfile** that follows best practices. Here's the corrected version of the **Dockerfile** with explanations for each improvement:

```dockerfile
# Use specific version of base image
FROM python:3.11-slim

# Use LABEL instead of deprecated MAINTAINER
LABEL maintainer="testuser@test.com"

# Set working directory
WORKDIR /app

# Copy only requirements file first to leverage Docker cache
COPY requirements.txt .

# Combine RUN commands, pin versions, clean cache, and create non-root user
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl=7.88.1-10 && \
    pip install --no-cache-dir -r requirements.txt && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    groupadd -r appuser && \
    useradd -r -g appuser appuser && \
    chown -R appuser:appuser /app

# Copy application code
COPY --chown=appuser:appuser . .

# Switch to non-root user
USER appuser

# Document the port that will be exposed
EXPOSE 8000/tcp

# Use JSON notation for CMD
CMD ["python3", "app.py"]
```

By following [Hadolint](https://github.com/hadolint/hadolint) recommendations, we've significantly improved our **Dockerfile**. The optimized version is:

- More secure (non-root user, pinned versions).
- More efficient (reduced layers, optimized cache).
- More maintainable (clear structure, documented choices).
- More reliable (specific versions, reproducible builds).

Remember that [Hadolint](https://github.com/hadolint/hadolint) is just one tool in your Docker development toolkit. While it helps catch common issues and enforce best practices, it's important to also consider your specific use case and requirements when building Docker images.

