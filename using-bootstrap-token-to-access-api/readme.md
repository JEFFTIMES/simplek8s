# checking the enablement of the bootstrap token authentication

make sure the following flag is set to the commands field in the kube-apiserver.yaml

    - --enable-bootstrap-token-auth=true


> refer [configure kube-apiserver.yaml of the cluster created by docker-desktop](https://stackoverflow.com/questions/64758012/location-of-kubernetes-config-directory-with-docker-desktop-on-windows) for the way to configure api-server.


# creating the bootstrap token

- creating the [manifest](./secret-bootstrap-token.yaml) of the Secret to create the bootstrap token


    apiVersion: v1
    kind: Secret
    metadata:
      # Name MUST be of form "bootstrap-token-<token id>"
      name: bootstrap-token-07401b
      namespace: kube-system

    # Type MUST be 'bootstrap.kubernetes.io/token'
    type: bootstrap.kubernetes.io/token
    stringData:
      # Human readable description. Optional.
      description: "the bootstrap token generated by Jeff manually."

      # Token ID and secret. Required.
      token-id: 07401b
      token-secret: f395accd246ae52d

      # Expiration. Optional.
      expiration: 2023-03-10T03:22:11Z

      # Allowed usages.
      usage-bootstrap-authentication: "true"
      usage-bootstrap-signing: "true"

      # Extra groups to authenticate the token as. Must start with "system:bootstrappers:"
      auth-extra-groups: system:bootstrappers:worker,system:bootstrappers:ingress


> bootstrap token conform to the specification `xxxxxx.zzzzzzzzzzzzzzzz`: 6 digits of token-id and 16 digits of token-secret join with a '.'.
> the creation of the bootstrap token generate a user in the format of `system:bootstrap:<token-id>` 


- passing it to the API server

    $ kubectl apply -f secret-bootstrap-token.yaml



# create the Role and RoleBinding for the user


- creating the RoleBinding for the user


    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: rb-for-user-bootstrap
      namespace: project-k
    subjects:
    # You can specify more than one "subject"
    - kind: User
      name: system:bootstrap:07401b       # "name" is case sensitive
    roleRef:
      # "roleRef" specifies the binding to a Role / ClusterRole
      kind: Role #this must be Role or ClusterRole
      name: role-dev-in-ns-project-k # this must match the name of the Role or ClusterRole you wish to bind to
      apiGroup: rbac.authorization.k8s.io



# creating a credentials entry and context for the bootstrap user.

- creating the credentials entry

    $ kubectl config set-credentials system:bootstrap:07401b --token=$(echo 07401b.f395accd246ae52d | base64)

- checking the credentials

    $ kubectl config view --raw -o jsonpath='{.users[?(@.name=="system:bootstrap:07401b")].user.token}' | base64 -d
    07401b.f395accd246ae52d

- creating the context

    $ kubectl config set-context bootstrap-user-07401b \
              --cluster=docker-desktop \
              --namespace=project-k \
              --user=system:bootstrap:07401b

# Accessing the API Server using `kubectl`

    $ kubectl --token 07401b.f395accd246ae52d get pods --namespace project-k

    
# Accessing the API Server using restful api

    $ curl  --cacert ca.crt \
            --header "Authorization: bearer 07401b.f395accd246ae52d" \
            -X GET https://localhost:6443/api/v1/namespaces/project-k/pods
