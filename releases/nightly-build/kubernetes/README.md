## edgexfoundry/developer-scripts/releases/nightly-build/kubernetes

Deploy EdgeX's nightly build containers in Kubernetes.  Instructions assume a running Kubernetes cluster and local 
    access to the kubectl executable.

1. Change into this directory and start EdgeX.

    ```kubectl create -k .```
    
1. List configured ingress objects.

    ```kubectl get ingress```
    
1. Repeat prior step until all EdgeX services have a populated ADDRESS field.  For example:

    ```
    NAME                          HOSTS              ADDRESS          PORTS   AGE
    edgex-core-consul             consul.k8s         192.168.150.76   80      2m11s
    edgex-core-command            command.k8s        192.168.150.76   80      2m11s
    edgex-core-data               data.k8s           192.168.150.76   80      2m11s
    edgex-core-metadata           metadata.k8s       192.168.150.76   80      2m10s
    edgex-export-client           client.k8s         192.168.150.76   80      2m10s
    edgex-export-distro           distro.k8s         192.168.150.76   80      2m10s
    edgex-support-logging         logging.k8s        192.168.150.76   80      2m10s
    edgex-support-notifications   notifications.k8s  192.168.150.76   80      2m10s
    edgex-support-rulesengine     rulesengine.k8s    192.168.150.76   80      2m10s
    edgex-support-scheduler       scheduler.k8s      192.168.150.76   80      2m10s
    ```

1. Update host machine's hosts file to map HOSTS domain to ADDRESS.

1. Access services via domain name; i.e. ```http://metadata.k8/api/v1/device```

1. To remove all EdgeX-related objects, change into this directory and stop EdgeX.

    ```kubectl delete -k .``` 
    
### Notes

- edgex-mongo and edgex-support-rulesengine use 1.0.0 (Edinburgh) images.  There were not 1.1.0 versions of the images 
    for these services in Nexus at the time the manifests were created.