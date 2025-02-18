+++
date = '2025-02-16T19:58:52+01:00'
draft = false
title = 'Docker Image Security Scanning With Trivy'
+++

## Overview

Security in container images has become one of the most critical aspects of maintaining a secure and resilient containerized environment. As more organizations embrace containerization for its scalability, portability, and efficiency, the need to ensure that the containers are secure has grown exponentially. Container images, when built improperly or left unmonitored, can become a significant vector for cyberattacks, leading to vulnerabilities that expose systems and data to threats.

[Trivy](https://trivy.dev/latest/) is an open-source vulnerability scanner specifically designed to address this concern. [Trivy](https://trivy.dev/latest/) simplifies the process of scanning container images, as well as other artifacts such as Helm charts, Kubernetes resources or Git repositories, for known vulnerabilities. Analyzing the underlying components of an image, including operating system packages, libraries, and dependencies, and cross-references them against an up-to-date vulnerability database.

What sets Trivy apart is its ease of use and speed. It’s designed to work out of the box with minimal setup, enabling users to quickly identify and mitigate security risks without the complexity often associated with other security tools. This simplicity, coupled with its depth of scanning and broad ecosystem support, has made Trivy a go-to tool in the DevOps community for integrating vulnerability scanning into CI/CD pipelines.

## Installation

If you don’t have [Trivy](https://trivy.dev/latest/) installed yet, you can install on macOS with Homebrew:

```bash
brew install trivy
```

Once the installation is complete, verify that the [Trivy](https://trivy.dev/latest/) installation was successful by entering the following command in the terminal:

```bash
trivy --version
Version: 0.59.1
```

## Scanning for vulnerabilities

When you run the [Trivy](https://trivy.dev/latest/) command to scan for vulnerabilities in your container images, [Trivy](https://trivy.dev/latest/) first ensures that it has the most up-to-date vulnerability database by downloading the relevant data. It then performs a thorough comparison, scanning the components of your container image—including operating system packages, libraries, and dependencies—against the vulnerabilities listed in the database. This allows [Trivy](https://trivy.dev/latest/) to identify and report any known security risks within the image.

[Trivy](https://trivy.dev/latest/) categorizes the identified vulnerabilities into different severity levels to help you prioritize your response:

- **Critical:** These vulnerabilities represent the **highest level of risk** and need to be addressed immediately. They typically allow attackers to gain administrative control over the system, which can lead to full compromise of the container or even the underlying host. Critical vulnerabilities pose a significant threat to the integrity and security of your environment, and they should be fixed without delay to prevent severe security breaches.

- **High:** High-risk vulnerabilities are serious and should be prioritized promptly. While they may not give attackers full control over the system, they increase the likelihood of data leakage, unauthorized access, or other forms of exploitation. These vulnerabilities could potentially result in sensitive data being exposed or allow attackers to manipulate the system in ways that are damaging to your organization's security posture. It's crucial to resolve these as quickly as possible to minimize potential damage.

- **Medium:**  Medium-level vulnerabilities may not present an immediate or catastrophic threat but still warrant attention. These types of issues could result in the unavailability of the system or application for users, potentially leading to service disruptions or downtime. While not as urgent as critical or high risks, they should be addressed within a reasonable timeframe to maintain system stability and user experience.

- **Low:** Low-risk vulnerabilities generally have a limited impact on system security and typically don’t pose an immediate threat. These vulnerabilities can often be addressed during regular maintenance or as part of routine updates and patching. While they should not be ignored, they do not require immediate action and can be resolved during less urgent security or system updates.

By classifying vulnerabilities into these categories, [Trivy](https://trivy.dev/latest/) provides a clear and actionable way to prioritize remediation efforts based on the severity of each issue and helps teams take appropriate action, ensuring that the most critical vulnerabilities are patched first while allowing for a more measured approach to lower-severity risks.

## Key components Trivy scans in container images

As we have seen in previous sections, [Trivy](https://trivy.dev/latest/) provides comprehensive scanning for container images, helping you identify vulnerabilities in various key components. Here are the main elements [Trivy](https://trivy.dev/latest/) inspects during its scan:

- **Package Managers and Dependencies:** [Trivy](https://trivy.dev/latest/) is capable of scanning multiple package managers, such as ```apt```, ```yum```, ```apk```, and ```npm```. Checking for vulnerabilities across a wide range of software dependencies, regardless of the package manager used in your container image.

- **Cross-Platform Scanning:** Whether your image is designed for Linux or Windows, [Trivy](https://trivy.dev/latest/) can scan vulnerabilities in both types. This flexibility ensures that no matter which platform you're deploying on, your images are adequately checked for security risks.

- **Diverse Image Formats:** [Trivy](https://trivy.dev/latest/) can scan images in various formats, including Docker images, tar archives, and filesystems. Whether your images are stored locally or in a specific archive format, Trivy can analyze them for vulnerabilities.

- **Environments and Deployments:** [Trivy](https://trivy.dev/latest/) is not limited to scanning images on local Docker containers; it also works with images running in Kubernetes pods and other environments. This allows you to scan images across the entire deployment pipeline, whether they are running in development, staging, or production environments.

## Scanning Docker images.

The **Alpine Linux** image is a minimal, security-focused Linux distribution commonly used as a base image for creating lightweight Docker containers. However, even minimal images like Alpine can contain vulnerabilities, making it crucial to scan them regularly to ensure they don't introduce security risks into your application.

To demonstrate the power of [Trivy](https://trivy.dev/latest/) lets scan the ```alpine:3.18.12``` Docker image, the first step is to pull it from (Docker Hub](https://hub.docker.com/). If you don’t have the image locally yet, you'll need to pull it so you can perform the vulnerability scan. Running the following command will download the ```alpine:3.18.12``` image to your local machine:

```bash
docker pull alpine:3.18.12
3.18.12: Pulling from library/alpine
44cf07d57ee4: Download complete
Digest: sha256:de0eb0b3f2a47ba1eb89389859a9bd88b28e82f5826b6969ad604979713c2d4f
Status: Downloaded newer image for alpine:3.18.12
docker.io/library/alpine:3.18.12
```

Once the image has been successfully pulled, you're ready to proceed with the vulnerability scanning process using [Trivy](https://trivy.dev/latest/) to identify any potential security issues that may exist within the image. To make the analysis shorter, I'll focus the results solely on critical vulnerabilities, you can use the following command in Trivy:

```bash
trivy image --severity CRITICAL alpine:3.13.2
2025-02-17T16:30:02+01:00	INFO	[vuln] Vulnerability scanning is enabled
2025-02-17T16:30:02+01:00	INFO	[secret] Secret scanning is enabled
2025-02-17T16:30:02+01:00	INFO	[secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2025-02-17T16:30:02+01:00	INFO	[secret] Please see also https://aquasecurity.github.io/trivy/v0.59/docs/scanner/secret#recommendation for faster secret detection
2025-02-17T16:30:03+01:00	INFO	Detected OS	family="alpine" version="3.13.2"
2025-02-17T16:30:03+01:00	INFO	[alpine] Detecting vulnerabilities...	os_version="3.13" repository="3.13" pkg_num=14
2025-02-17T16:30:03+01:00	INFO	Number of language-specific files	num=0
2025-02-17T16:30:03+01:00	WARN	This OS version is no longer supported by the distribution	family="alpine" version="3.13.2"
2025-02-17T16:30:03+01:00	WARN	The vulnerability detection may be insufficient because security updates are not provided

alpine:3.13.2 (alpine 3.13.2)

Total: 4 (CRITICAL: 4)

┌──────────────┬────────────────┬──────────┬────────┬───────────────────┬───────────────┬──────────────────────────────────────────────────────────────┐
│   Library    │ Vulnerability  │ Severity │ Status │ Installed Version │ Fixed Version │                            Title                             │
├──────────────┼────────────────┼──────────┼────────┼───────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ apk-tools    │ CVE-2021-36159 │ CRITICAL │ fixed  │ 2.12.1-r0         │ 2.12.6-r0     │ libfetch: an out of boundary read while libfetch uses strtol │
│              │                │          │        │                   │               │ to parse...                                                  │
│              │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2021-36159                   │
├──────────────┼────────────────┤          │        ├───────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ libcrypto1.1 │ CVE-2021-3711  │          │        │ 1.1.1j-r0         │ 1.1.1l-r0     │ openssl: SM2 Decryption Buffer Overflow                      │
│              │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2021-3711                    │
├──────────────┤                │          │        │                   │               │                                                              │
│ libssl1.1    │                │          │        │                   │               │                                                              │
│              │                │          │        │                   │               │                                                              │
├──────────────┼────────────────┤          │        ├───────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ zlib         │ CVE-2022-37434 │          │        │ 1.2.11-r3         │ 1.2.12-r2     │ zlib: heap-based buffer over-read and overflow in inflate()  │
│              │                │          │        │                   │               │ in inflate.c via a...                                        │
│              │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2022-37434                   │
└──────────────┴────────────────┴──────────┴────────┴───────────────────┴───────────────┴──────────────────────────────────────────────────────────────┘
```

## Integrating Trivy with GitHub Actions

Automating container image vulnerability scanning as part of your CI/CD pipeline is crucial for maintaining a secure environment. [GitHub Actions](https://github.com/features/actions) provides a great way to automate the scanning process using [Trivy](https://trivy.dev/latest/).

Here’s how you can integrate **Trivy** into your [GitHub Actions](https://github.com/features/actions) workflow:

Start by creating a `.github/workflows/trivy-scan.yml` file in your repository. This file will define the steps for scanning your container images with [Trivy](https://trivy.dev/latest/) every time a pull request is created or when changes are pushed to the repository. 

This integration ensures that every time code is updated, [Trivy](https://trivy.dev/latest/) will automatically scan the associated Docker images for vulnerabilities, helping to catch any security issues early in the development cycle.


```yaml
name: trivy-scanning

on:
  push:
    branches: [ "master" ]
  pull_request:
    branches: [ "master" ]

permissions:
  contents: write  # Adjusted permission scope

jobs:
  trivy-scanning-job:
    name: trivy-sec-scan
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Run Trivy vulnerability scanner in repo mode
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          ignore-unfixed: true
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'HIGH,CRITICAL'

      - name: Upload Trivy scan results to GitHub Security tab
        uses: github/codeql-action/upload-sarif@v1
        with:
          sarif_file: 'trivy-results.sarif'

      - name: Build an image from Dockerfile
        run: |
          docker build -t docker.io/devops-counsel/py-app:${{ github.sha }} .

      - name: Run Trivy vulnerability scanner on Docker image
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'docker.io/devops-counsel/py-app:${{ github.sha }}'
          format: 'table'
          vuln-type: 'os,library'
          severity: 'CRITICAL,HIGH'

```

After the workflow is set up, [GitHub Actions](https://github.com/features/actions) will run [Trivy](https://trivy.dev/latest/) on your Docker images each time a pull request is opened or code is pushed. You can view the results of the vulnerability scan in the Actions tab of your GitHub repository.

If any vulnerabilities are found, the job will fail, and you’ll see the specific security issues and their severity levels from [Trivy](https://trivy.dev/latest/) in the logs. You can then address the vulnerabilities in your Docker image and push the necessary changes to your repository.

This process helps ensure that your container images are always secure and free from critical vulnerabilities before they are deployed to production.

