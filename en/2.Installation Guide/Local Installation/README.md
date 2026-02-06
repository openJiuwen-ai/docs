# Local Installation Guide

Local installation provides multiple installation methods, suitable for developers who need to modify code, perform secondary development, or require flexible configuration.

## Installation Methods

Local installation provides two methods:

* **Method 1: Using One-Click Installation Script**: Automatically completes most installation and configuration work, simplifying the installation process and suitable for rapid deployment.
* **Method 2: Manual Installation** (Not Recommended): Requires manual installation and configuration of all dependency services, suitable for developers who need flexible configuration adjustments.

Each system's installation documentation contains detailed instructions for these methods. Please select the corresponding installation guide based on your system:

* [Windows Installation](./Windows_Installation.md)
* [macOS Installation](./MacOS_Installation.md)
* [Linux Installation](./Linux_Installation.md)

## Prerequisites

* Familiar with Linux/Unix system operations
* Understanding of service configuration and management
* Experience with network and proxy configuration
* Ability to independently resolve issues during installation

## Required Dependency Services

* MySQL database
* MinIO object storage
* Milvus vector database
* etcd distributed key-value store
* Other project-specific dependency services

> **Note**: Local installation has a higher entry barrier and requires users to have strong technical background. If there are no special requirements, it is recommended to prioritize using [Docker Quick Installation](../Install%20via%20Docker/README.md).