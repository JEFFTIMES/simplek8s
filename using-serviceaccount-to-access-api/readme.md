# creating a service account 

    $ kubectl create serviceaccount sa-project-k

# retrieving the token for the service account

    $ kubectl create token sa-project-k --namespace project-k 


# accessing the api server

    $ curl --cacert ca.crt \
            --header "Authorization: Bearer $(kubectl create token sa-project-k --namespace project-k)" \
            -X GET https://kubernetes.docker.internal:6443/api/v1  

# binding a role to the service account

[role binding for the service account](./rb-for-user-sa.yaml)


# accessing the resources in the namespace project-k

- restful api
    $ curl --cacert ca.crt \
            --header "Authorization: Bearer $(kubectl create token sa-project-k --namespace project-k)" \
            -X GET https://kubernetes.docker.internal:6443/api/v1/namespaces/project-k/pods


- kubectl cli
    $ kubectl --token $(kubectl create token sa-project-k --namespace project-k) get pods --namespace project-k