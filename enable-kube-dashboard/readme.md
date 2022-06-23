# Check the enablement of the RBAC

run the following command to check if the `rbac.authorization.k8s.io/v1` is within the output.

    $ kubectl apiversions

# Install containers for the dashboard

    dashboard/v2.5.0/aio/deploy/recommended.yaml


# Create a `ServiceAccount` for the admin user (to get the token to)

**enable-kube-dashboard/serviceaccount-admin-user.yaml**

    apiVersion: v1
    kind: ServiceAccount
    metadata: 
      name: admin-user
      namespace: kubernetes-dashboard
  

# Check the existence of `cluster-admin` `ClusterRole`

    $ kubectl get clusterroles cluster-admin -o yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      annotations:
        rbac.authorization.kubernetes.io/autoupdate: "true"
      creationTimestamp: "2022-06-12T05:34:49Z"
      labels:
        kubernetes.io/bootstrapping: rbac-defaults
      name: cluster-admin
      resourceVersion: "78"
      uid: 909abdaf-9ef1-4da1-b80c-9cc3e0ac6f64
    rules:
    - apiGroups:
      - '*'
      resources:
      - '*'
      verbs:
      - '*'
    - nonResourceURLs:
      - '*'
      verbs:
      - '*'


# Create the `RoleBinding` for the admin user

**rolebinding-admin-user.yaml**

    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata: 
      name: rolebinding-admin-user
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
      - kind: ServiceAccount
        name: admin-user
        namespace: kubernetes-dashboard


# Create a token for the admin user

    $ kubectl --namespace kubernetes-dashboard create token admin-user
    eyJhbGciOiJSUzI1NiIsImtpZCI6Iko0bnNYOTZJWkJVU2JRQ3E4ZWFKV053bGlpM1huQy1IMWY2R2RzZlNmcTgifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNjU2MDEzNTYxLCJpYXQiOjE2NTYwMDk5NjEsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhZG1pbi11c2VyIiwidWlkIjoiNTEwOTYyZjYtMjNhYi00NTgyLTg5ZjUtMWMxNzBiNmYwZmYyIn19LCJuYmYiOjE2NTYwMDk5NjEsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDphZG1pbi11c2VyIn0.y25-ghj8ZszG8LN_GqJBAaBljaNcOjaLRH8NhvUNcsCqk-_MnLrSNRZcYvvOLi9_Ge8C4OMYBcfg03BnsiuZ9D9iLles3zTvLevandM9QsvJStImwi6g4HD5h_T2DNHPXCJvCSGsdZGmy9b_AWduM_G_eI6gxNp9knpyLRyhLajnSQaPF2Kyqn0Q43G5QQa3vz2OYCxc2TgWu8psOLIv7zKK3tukC8U-i-VbA62k4IOKEzoQHUc08tGRBtVMIL9Q945D0iasu1Sh13GmFozbBXJ0iMIFllsa6ewVuntJuAQAAn9QbO8JdYkcoAudNejoX_iuj90AFfc0xGEQ1JOSRA


# Proxy the api server endpoint

    $ kubectl proxy




# Access the endpoint of the api server  

    http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/



# Login in with the token

* select the token button.

* paste the token created in the field and press the `sign in` button.


