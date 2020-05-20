# Deploying Containerized Applications

Annotations from Red Hat free course Deploying [Containerized Applications Technical Overview DO080](https://www.redhat.com/rhtapps/promo-do080)

## Overview of Container Technology

- Traditional architecture, application closely coupled with OS and its dependencies
- Users needed a fast and lightweight way to spin up VMs
- Containers are a set of one or more processes that are isolated from the of the system.
    - Low Hardware Footprint
    - Quick and Reeusable Deployment
    - Environment Isolation
    - Portable

## Overview of Container Architecture

- Existing technologies within Linux kernel
    - Namespaces
    - cgroups (limit resources)
    - Seccomp
    - SELinux (protect process from each other and host system)
- Terms
    - Containers
    - Image (Template for containers)
    - Image Repository (public or private registries where images are stored)
- Podman is an open source tool for managing containers

## Overview of Kubernetes and OpenShift

- As the number of containers grows, the work of manually starting them rises exponentially
- Users needed a way to orchestrate containers
- Orchestration
- Scheduling
    - scale up and scale down the number of containers
- Isolation
    - Prevent Cascade failures
- Kubernetes
    - Service discovery and Load balacing
    - Horizontal scaling
    - Self-healing
    - Automated rollout and rollback
    - Secrets and configuration management
    - Operators
- Openshit
    - Build on top of Kubernetes
    - Integrated developer workflow
    - Routes
    - Metrics and logging
    - Unified UI

## Provisioning a Containerized Database Server (and demonstration)

 - Fetching Container images
    
    ```$ podman search rhel```
    
    ```$ podman pull rhel```

- Container Image Name Syntax

    `registry_name/user_name/image_name:tag`

- Running Containers

    `$ podman run rhel7:7.5 echo "Hello world"`

- Container options
    - Use `-d` to running a container in background
    - Name containers with `--name`
    - `-t` for Pseudo Terminal and `-i` for Interactive attached to STDIN
    - `-e` to pass Environment variables when start a container

- Demostration
    - Running mysql container

    ```
        $ sudo podman run --name mysql-basic \
        -e MYSQL_USER=user1 -e MYSQL_PASSWORD=mypass \
        -e MYSQL_DATABASE=items -e MYSQL_ROOT_PASSWORD=r00t \
        -d rhscl/mysql-57-rhel7:5.7-3.14
    ```

    - Check running container

        `$ sudo podman ps`

    - Interactive shell within the container

        `$ sudo podman exec -it mysql-basic /bin/bash`


Building Custom Container Images with Dockerfiles (and demonstration)
Creating Basic Kubernetes and OpenShift Resources (and demonstration)
Creating Applications with the Source-to-Image Facility (and demonstration)
Creating Routes (and demonstration)
Creating Applications with the OpenShift Web Console (and demonstration)