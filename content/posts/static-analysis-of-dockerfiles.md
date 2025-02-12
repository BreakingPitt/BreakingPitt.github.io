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

[Hadolint](https://github.com/hadolint/hadolint) can be installed in different ways, depending on your operating system. Here’s how to do it using a Mac computer:

```bash
brew install hadolint
```

