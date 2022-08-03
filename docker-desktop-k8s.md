
## 1. configure `kube-apiserver.yaml` or `kube-controller-manager.yaml`

These yaml files should be existed in the `Pod` api-server within `Namespace` kube-system.
But it is not easy to login to the container with general methods like `kubectl exec -it ...`. 

As for the reconfiguration of `kube-apiserver.yaml` with Docker Desktop

You need to run following command:

    docker run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh

Above command will bring you to the container, allows you to run:

    vi /etc/kubernetes/manifests/kube-apiserver.yaml

to modify the configuration file, as well as allows you to make the files correspond to the configuration fields such as 

    --tls-cert-file=/run/config/pki/apiserver.crt


