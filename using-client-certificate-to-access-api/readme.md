# the user information

- username: jeff
- group: dev
- project: project-k
- private key: jeff.key
- csr: jeff.csr
- certificate: jeff.cert
  

# the process

1. Creating the private key for the user.
2. Creating the Certificate Signing Request for the user.
3. Submitting the Certificate Signing Request (CSR) to the API server for approval.
4. Approving the Certificate Signing Request and Issuing certificate to the user.
5. Retrieving the client Certificate and the CA certificate.
6. Adding user credentials and context to the kubeconfig for the user to use `kubectl`.
7. Using the kubeconfig to access the API server.
8. Creating a role for the user.
9. Create a RoleBinding for the role.
10. Accessing API server with the proper permissions.

## 1. Creating the private key for the user

- Using `openssl` to create the private key.

    $ openssl genrsa -out jeff.key


- Examining the private key for the user.

    $ openssl pkey -in jeff.key -text --noout


## 2. Creating the Certificate Signing Request for the user

- Generating the Certificate Signing Request

    $ openssl req -new -key jeff.key -out jeff.csr -subj "/CN=jeff/O=dev"

- Checking the Certificate Signing Request

    $ openssl req -text -in jeff.csr -noout

## 3. Submitting the Certificate Signing Request (CSR) to the API server for approval

- Encoding the Certificate Signing Request to a base64 string

    $ base64 jeff.csr | tr -d "\n" > jeff.csr.bs64

- Creating a CSR object yaml file

    $ cat << EOF > jeff.csr.yaml
    apiVersion: certificates.k8s.io/v1
    kind: CertificateSigningRequest
    metadata:
      name: jeff
    spec:
      request: $(base64 jeff.csr | tr -d '\n')
      signerName: kubernetes.io/kube-apiserver-client
      expirationSeconds: 8640000  # 100 days
      usages:
      - client auth
    EOF

- Applying the Certificate Signing Request to the API server

    $ kubectl apply -f jeff.csr.yaml 


- Checking the Certificate Signing Request object

    $ kubectl get certificatesigningrequests.certificates.k8s.io


## 4. Approving the Certificate Signing Request

- Approving the Certificate Signing Request

    $ kubectl certificate approve jeff

- Checking the status of the Certificate Signing Request

    $ kubectl get csr

- Checking the signed Certificate

    $ kubectl get csr -jeff -o jsonpath='{.status.certificate}'

## 5. Retrieving the signed client Certificate

    $ kubectl get csr jeff -o jsonpath='{.status.certificate}' | base64 -d > jeff.cert

## 6. Retrieving the Certificate of the cluster CA

    $ kubectl config view --raw -o jsonpath='{.clusters[0].cluster.certificate-authority-data}' | base64 -d > ca.cert

## 7. Configuring the user credentials and context into the kubeconfig

    $ kubectl config set-credentials jeff \
    --client-certificate=jeff.cert \
    --client-key=jeff.key \
    --embed-certs 


    $ kubectl config set-context jeff \
    --cluster=$(kubectl config view -o jsonpath='{.clusters[0].name}') \
    --user=jeff \

## 8.  Switching between users 

- to a user jeff
    $ kubectl config use-context jeff

- back to docker-desktop
    $ kubectl config use-context docker-desktop


## 9. Creating a Role with permissions.

- manifest: 

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      namespace: project-k
      name: role-dev-in-ns-project-k
    rules:
    - apiGroups: ["", "apps", "batch", "storage.k8s.io"] # "" indicates the core API group
      resources: ["pods", "services", "deployments", "job", "configmaps", "secrets", "volumes"]
    verbs: ["get", "watch", "list", "create", "update", "delete", "rollback"]


- applying:

    $ kubectl apply -f role-dev-in-ns-project-k.yaml

## 10. Creating a RoleBinding to bind the user to the role.

- manifest: 

    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: rb-for-role-dev
      namespace: default
    subjects:
    # You can specify more than one "subject"
    - kind: User
      name:  jeff                          # "name" is case sensitive
      namespace: project-k
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      # "roleRef" specifies the binding to a Role / ClusterRole
      kind: Role #this must be Role or ClusterRole
      name: role-dev-in-ns-project-k      # this must match the name of the Role or ClusterRole you wish to bind to
      apiGroup: rbac.authorization.k8s.io

- applying: 
  
    $ kubectl apply -f rb-for-role-dev.yaml    


> the kind of Group does not work for the `subjects.kind` field of a RoleBinding, even the groups information of the user was specified in the client certificate through the subject parameter '/O=group1/O=group2'.
> As of Kubernetes 1.4, client certificates can also indicate a user's group memberships using the certificate's organization fields. 

## 11. Accessing the API server.

- switching user

    $ kubectl config use-context jeff 

- getting pods through `kubectl` cli

    $ kubectl get pods -n project-k   

- getting deployments through Restful API 

    $ curl  --cacert ca.cert \
            --cert jeff.cert \
            --key jeff.key \
            -X GET https://kubernetes.docker.internal:6443/apis/apps/v1/namespaces/project-k/deployments/
