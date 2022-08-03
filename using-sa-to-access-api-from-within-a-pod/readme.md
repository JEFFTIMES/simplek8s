 
# Purpose
Creating a Role and RoleBinding for the default ServiceAccount of the Namespace project-k, through which to allow the default ServiceAccount accessing the API Server from within a curlpod.

# Crafting a `Role` and a `RoleBinding` manifest

- Authorizing operational permissions on the resources to the `Role`. 
- In this case, all actions on any resources under any apiGroups are granted to the default `ServiceAccount`.

    rules:
    - apiGroups: ["*"]
      resources: ["*"]
      verbs: ["*"]


# Creating the `Role` and `RoleBinding` objects

    $ kubectl apply -f role-for-curl-working-in-a-pod.yaml


# Checking the creations
    $ kubectl get roles.rbac.authorization.k8s.io --namespace project-k

    $ kubectl get rolebindings.rbac.authorization.k8s.io --namespace project-k

# Creating `curl.yaml` contains a `Deployment` object to deploy the pod

# Deploying the `Pod`

    $ kubectl apply -f curl.yaml


# Shelling into the `Container`

    $  kubectl exec -it --namespace project-k \
       $(kubectl get pods --namespace project-k -o=jsonpath='{.items[0].metadata.name}') \
       sh

# Using the default `ca.crt` and `token` to access the API Server

## set environment variables

- `$ APISERVER=https://kubernetes.default` 
- `$ SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount`
- `NAMESPACE=$(cat ${SERVICEACCOUNT}/namespace)`
- `TOKEN=$(cat ${SERVICEACCOUNT}/token)`
- `CACERT=${SERVICEACCOUNT}/ca.crt`
  

## Accessing APIs through the API paths

- list pods (apiGroup: core/v1 (api/v1))

    $ curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/project-k/pods

- list deployments (apiGroup: apis/apps/v1)
  
    $ curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/apis/apps/v1/namespaces/project-k/deployments


    