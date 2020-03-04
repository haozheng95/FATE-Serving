### Building FATE Images

This document provides guidance on how to build Docker images for FATE-Serving from source code. 

If you are a user of FATE-Serving, you usually DO NOT need to build FATE-Serving from source code. Instead, you can use the pre-built docker images for each release of FATE-Serving. It saves time and greatly simplifies the deployment process of FATE-Serving. Refer to  [KubeFATE](https://github.com/FederatedAI/KubeFATE) for more details.

#### Prerequisites
To build the Docker images of FATE-Serving components, the host must meet the following requirements:

1. A Linux host
2. Docker 18+
3. The host can access the Internet.

#### Building images of FATE-Serving

##### Configuration
Before building the images, the `.env` file must be configured, so that the version of codebase can be easily identified. 

When images are built, images have the naming format as follows:

```
<PREFIX>/<image_name>:<TAG>
```

**PREFIX**: the namespace on the registry's server, it will be used to compose the image name.  
**TAG**: tag used for component images.

A sample of `.env` is as follows:
```bash
PREFIX=federatedai
TAG=1.3.0-release
```
**NOTE:** 
If the FATE-Serving images will be pushed to a registry server, the above configuration assumes the use of Docker Hub. If a local registry (e.g. Harbor) is used for storing images, change the `PREFIX` as follows:

```bash
PREFIX=<ip_address_of_registry>/federatedai
```

##### Running the build script
After configuring `.env`, use the following command to build all images:
```bash
$ bash build_cluster_docker.sh all
```

The command creates the base images and then the component images. After the command finishes, all images of FATE-Serving should be created. Use `docker images` to check the newly generated images:
```
REPOSITORY                            TAG  
federatedai/serving-server            <TAG>
federatedai/serving-proxy             <TAG>
```

##### Pushing images to a registry (optional)
To share the docker images with multiple nodes, images can be pushed to a registry (such as Docker Hub or Harbor registry).

Log in to the registry first: 

```bash
# for Docker Hub
$ docker login -u username 
```
or
```bash
# for Harbor or other local registry
$ docker login -u username <URL_of_Registry>
```

Next, push images to the registry. Make sure that the `PREFIX` setting in `.env` points to the correct registry service: 

```bash
$ bash build_cluster_docker.sh push
```


##### Package the docker images for offline deployment (optional)

Some environemts may not have access to the Internet. In this case, FATE-Serving's docker images can be packaged and copied to these environments for offline deployment.

On the machine with all FATE-Serving docker images available, use the following commands to export and package images:
```bash
$ docker save $(docker images | grep federatedai| grep -v -E "base|builder" | awk '{print $1":"$2}') -o fate-serving.images.tar.gz
```

One tar file should be generated `fate-serving.images.tar.gz` . The formmer one contains all FATE-Serving images. 

Copy the two tar files to the targeted deployment machine which is not connected to the Internet. Log in to the machine and use the following command to load the images:
```bash
$ docker load -i fate-serving.images.tar.gz
```

Now the machine has all FATE-Serving images and is ready for deployment. 

### Deployment
To use docker images to deploy FATE-Serving by Docker Compose or Kubernetes, please refer to [KubeFATE](https://github.com/FederatedAI/KubeFATE) for more details.