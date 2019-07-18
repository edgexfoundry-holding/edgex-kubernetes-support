# edgexfoundry-holding/edgex-kubernetes-support

## Overview

This repository contains supporting files to run various current and historical versions of EdgeX in Kubernetes 
    (including K3s).

## Table of Contents

- [Kubernetes Environments for Local Experimentation/Development](#kubernetes-environments-for-local-experimentationdevelopment)
    - [Minikube](#minikube)
        - [Minikube on Windows](#minikube-on-windows)
        - [Minikube on MacOS](#minikube-on-macos)
    - [Docker Desktop for Windows](#docker-desktop-for-windows)
    - [Docker Desktop for Mac](#docker-desktop-for-mac)
    - [Running a K3s Cluster Inside Docker](#running-a-k3s-cluster-inside-docker)
- [Nightly Build](#nightly-build)
    - [Kubernetes](#kubernetes-for-nightly-build)
    - [Helm Chart](#helm-chart-for-nightly-build)
- [Edinburgh Release](#edinburgh-release)
    - [Kubernetes](#kubernetes-for-edinburgh-release)
    - [Helm Chart](#helm-chart-for-edinburgh-release)
- [Delhi Release](#edinburgh-release)
    - [Kubernetes/Helm Chart](#kuberneteshelm-chart-for-delhi-release)
- [California Release](#california-release)
    - [Kubernetes/Helm Chart](#kuberneteshelm-chart-for-california-release)
- [Barcelona Release](#barcelona-release)
    - [Kubernetes/Helm Chart](#kuberneteshelm-chart-for-barcelona-release)
   
## Kubernetes Environments for Local Experimentation/Development
    
### Minikube

Minikube provides a single-node Kubernetes cluster that you can use to deploy EdgeX for experimentation or 
    development.  Supplied instructions and configuration was tested with Minikube version 1.14.

#### Minikube on Windows

1. Install [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/).

1. Start Minikube (following leverages Hyper-V):

    ```./minikube start --memory=8192 --cpus=4 --alsologtostderr --vm-driver hyperv --hyperv-virtual-switch=minikube-switch```

1. [Enable ingress add-on](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/): 

    ```minikube addons enable ingress```
    
#### Minikube on MacOS

1. Install [Hyperkit](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#hyperkit-driver)

1. Install [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/).

1. Start Minikube:

    ```./minikube start --memory=8192 --cpus=4 --alsologtostderr --vm-driver hyperkit```

1. [Enable ingress add-on](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/): 

    ```minikube addons enable ingress```

### Docker Desktop for Windows

[Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/#kubernetes) versions 18.06 Stable (win70) and 
    higher include support for an optional single-node Kubernetes cluster.
    
As of this writing, the version of kubectl included in the Docker Desktop for Windows install is 1.10.  The manifest
    files and instructions provided in this repository assume kubectl version 1.14 or higher.  The latest version
    of kubectl can be [downloaded](https://kubernetes.io/docs/tasks/tools/install-kubectl/), installed, and subsequently 
    used to apply the supplied `kustomization.yml` files to the Docker Desktop for Windows-based single-node Kubernetes 
    cluster.

### Docker Desktop for Mac

[Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/#kubernetes) versions 18.06 Stable (mac70) and 
    higher include support for an optional single-node Kubernetes cluster.
    
As of this writing, the version of kubectl included in the Docker Desktop for Mac install is 1.10.  The manifest
    files and instructions provided in this repository assume kubectl version 1.14 or higher.  The latest version
    of [kubectl}(https://kubernetes.io/docs/tasks/tools/install-kubectl/) can be downloaded, installed, and subsequently 
    used to apply the supplied `kustomization.yml` files to the Docker Desktop for Mac-based single-node Kubernetes 
    cluster.
    
### Running a K3s Cluster Inside Docker

Follow the instructions [here](https://github.com/rancher/k3s#running-in-docker-and-docker-compose) to provision
    a K3s cluster inside Docker.  The provisioning process will create a `kubeconfig.yaml` which should be referenced
    in kubectl invocations (e.g. `kubectl apply --kubeconfig kubeconfig.yaml -k .`).
    
Provided [Helm](https://helm.sh/) Charts can also be deployed to K3s.  As with kubectl invocations, you'll need to reference the 
    provisioning-created `kubeconfig.yaml` in your helm invocation (e.g. 
    `helm --kubeconfig kubeconfig.yaml init`).  Unlike installing a Helm Chart on a local K8s cluster (such as a 
    Minikube or Docker Desktop-based Kubernetes instance), there are some additional considerations when installing 
    the Helm Chart on K3s:
    
*   [Role-based access control](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) (RBAC) must be properly 
    configured.  To grant super-user access to all service accounts cluster-wide (which isn't recommended but may be
    suitable for a local experimentation/development instance):
    
    ```
    kubectl create clusterrolebinding serviceaccounts-cluster-admin \
      --kubeconfig kubeconfig.yaml \
      --clusterrole=cluster-admin \
      --group=system:serviceaccounts
    ```
      
*   Add a command line parameter to explicitly specify the Helm Chart's Release Name; i.e. `--name fooRelease`
*   Add a command line parameter to explicitly specify kube-system as the target namespace; i.e. `--namespace kube-system`

A resulting Helm Chart install command might look something like:

    helm --kubeconfig ~/source/dell/k3s/kubeconfig.yaml --namespace kube-system --name foo install .
    
## Nightly Build 

### Kubernetes for Nightly Build

**This release of EdgeX is not fully [twelve-factor](https://12factor.net/) compliant and does not explicitly support
    high availability.** While EdgeX will run on Kubernetes, Kubernetes deployment objects for individual EdgeX services 
    should not have more than one replica running at any point in time. 

Available releases include:

* [`releases/nightly-build/kubernetes/kustomization.yaml`](https://github.com/edgexfoundry/developer-scripts/tree/master/releases/nightly-build/kubernetes/kustomization.yaml) 
    references service-specific Kubernetes manifest files that use the latest EdgeX nightly build container images from 
    the EdgeX Nexus repository .  This kustomization file does not include security or system management support.  See 
    the [README.md](https://github.com/edgexfoundry/developer-scripts/tree/master/releases/edinburgh/kubernetes/README.md) 
    for deployment details.

These manifest files use [Kustomize](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/) which 
    requires [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) version 1.14 or greater.
    
These manifest files limit volume mounts to those services that actually require volume access.  Unlike the 
    docker-compose files for this release (which use a separate Docker volume container), the manifest files mount 
    host based volumes as follows:
    
*   edgex-core-consul's `/consul/config` and `/consul/data` directories are is mapped to the host's 
    `/mnt/core-consul-volume` directory. 
*   edgex-mongo's `/data/db` directory is mapped to the host's `/mnt/mongo-volume` directory. 
*   edgex-support-logging's `/edgex/logs` directory is mapped to the host's `/mnt/support-logging-volume` directory. 

These manifest files work because they name containers using the same names found in the default configuration files 
    built into the Docker images. If you modify the container names in the manifest file, individual services will
    no longer be able to connect to one another.  Support for environment variable overrides of the configuration 
    built into the Docker images was introduced post-Edinburgh.  An example of leveraging environment variables to 
    override configuration in support of alternate container names can be found in the 
    [Nightly Build Helm Chart's](#helm-chart-for-nightly-build) templates.
    
The Nightly Build manifests and the Edinburgh Release manifests use the same names for corresponding Kubernetes 
    objects.  As a result, only one of these releases can be deployed on a given Kubernetes cluster at any one point in 
    time.  If you want to deploy the Edinburgh Release and the Nightly Build to the same Kubernetes cluster 
    simultaneously, you can do so by deploying the [Nightly Build via Helm](#helm-chart-for-nightly-build).

### Helm Chart for Nightly Build

**This release of EdgeX is not fully [twelve-factor](https://12factor.net/) compliant and does not explicitly support
    high availability.** While EdgeX will run on Kubernetes, Kubernetes deployment objects for individual EdgeX services 
    should not have more than one replica running at any point in time. 

Available releases include:

* [`releases/nightly-build/heml/nightly-build-no-security`](https://github.com/edgexfoundry/developer-scripts/tree/master/releases/nightly-build/helm/nightly-build-no-security) 
    contains a [Helm](https://helm.sh/) Chart that uses the latest EdgeX nightly build container images from the EdgeX 
    Nexus repository.  It does not include rules engine, security or system management support.  

## Edinburgh Release

### Kubernetes for Edinburgh Release

**This release of EdgeX is not fully [twelve-factor](https://12factor.net/) compliant and does not explicitly support
    high availability.** While EdgeX will run on Kubernetes, Kubernetes deployment objects for individual EdgeX services 
    should not have more than one replica running at any point in time. 

Available releases include:

* [`releases/edinburgh/kubernetes/kustomization.yaml`](https://github.com/edgexfoundry/developer-scripts/tree/master/releases/edinburgh/kubernetes/kustomization.yaml) 
    references service-specific Kubernetes manifest files that use EdgeX **Edinburgh release 1.0.0** container 
    images.  This kustomization file does not include security or system management support.  See the 
    [README.md](https://github.com/edgexfoundry/developer-scripts/tree/master/releases/edinburgh/kubernetes/README.md)
    for deployment details.

These manifest files use [Kustomize](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/) which 
    requires [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) version 1.14 or greater.
    
These manifest files limit volume mounts to those services that actually require volume access.  Unlike the 
    docker-compose files for this release (which use a separate Docker volume container), the manifest files mount 
    host based volumes as follows:
    
*   edgex-core-consul's `/consul/config` and `/consul/data` directories are is mapped to the host's 
    `/mnt/core-consul-volume` directory. 
*   edgex-mongo's `/data/db` directory is mapped to the host's `/mnt/mongo-volume` directory. 
*   edgex-support-logging's `/edgex/logs` directory is mapped to the host's `/mnt/support-logging-volume` directory. 

These manifest files work because they name containers using the same names found in the default configuration files 
    built into the Docker images. If you modify the container names in the manifest file, individual services will
    no longer be able to connect to one another.

The Nightly Build manifests and the Edinburgh Release manifests use the same names for corresponding Kubernetes 
    objects.  As a result, only one of these releases can be deployed on a given Kubernetes cluster at any one point in 
    time.  If you want to deploy the Edinburgh Release and the Nightly Build to the same Kubernetes cluster 
    simultaneously, you can do so by deploying the [Nightly Build via Helm](#helm-chart-for-nightly-build).

### Helm Chart for Edinburgh Release

There is no supporting [Helm](https://helm.sh/) Chart for the Edinburgh release. To support Helm's basic use case of 
    installing multiple instances of a chart, changes to EdgeX were made post-Edinburgh to support environment variable 
    overrides of the configuration built into the Docker images.  This enabled the use of release-specific container 
    names.  Thus, Helm Chart support is only possible in post-Edinburgh releases of EdgeX.

## Delhi Release

### Kubernetes/Helm Chart for Delhi Release

There are no supporting files for running the Delhi release in Kubernetes.

## California Release

### Kubernetes/Helm Chart for California Release

There are no supporting files for running the California release in Kubernetes.

## Barcelona Release

### Kubernetes/Helm Chart for Barcelona Release

There are no supporting files for running the Barcelona release in Kubernetes.