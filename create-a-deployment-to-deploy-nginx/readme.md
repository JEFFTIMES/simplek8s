# Deploying an nginx `Pod` with a `Service` of type `NodePort`

    $ kubectl apply -f deployment-nginx-http-nodeport.yaml

# Deploying an nginx `Pod` with a `Service` of type `LoadBalancer`

    $ kubectl apply -f deployment-nginx-http-load_balancer.yaml

# Deploying an nginx `Service` with HTTPS enabled

## 1. Creating a `Secret` with type TLS for nginx enabling the https

### Using openssl to generate the private key `ca.key` and the self-signed certificate `ca.crt` 

    $ openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout $(pwd)/ca-private.key \
    -out $(pwd)/ca-self-signed.crt \
    -subj "/CN=secured-nginx/O=dev"

> the common name of the certificate set as `CN=secured-nginx` should be used to set the name of the `Service` when creating the service for the nginx instance.

### Converting the key and the certificate to base64-encoded

    $ cat ca-private.key | base64 > base64-ca-private.key
    $ cat ca-self-signed.crt | base64 > base64-ca-self-signed.crt

### Creating a TLS type `Secret` from the generated key and certificate

secret-tls-nginx-https.yaml

    apiVersion: v1
    kind: Secret
    metadata: 
      namespace: project-k
      name: secret-tls-nginx-https
    type: kubernetes.io/tls 
    data:
      tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJBQ0NRRGdBV0VNOEE5N216QU5CZ2txaGtpRzl3MEJBUXNGQURBbU1SWXdGQVlEVlFRRERBMXoKWldOMWNtVmtMVzVuYVc1NE1Rd3dDZ1lEVlFRS0RBTmtaWFl3SGhjTk1qSXdOakkyTVRReE1qUXpXaGNOTWpNdwpOakkyTVRReE1qUXpXakFtTVJZd0ZBWURWUVFEREExelpXTjFjbVZrTFc1bmFXNTRNUXd3Q2dZRFZRUUtEQU5rClpYWXdnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFDa1RrS0pOaVh2S0w3dTFQVm0KOFI5MzJEN1c0dmw4L2E4c0JxNVhIaTUwUTlTeXdmMEVmc1J0ZTBIdTYra2dpZ2NEK2VFMy9WN2dWT1RUc3NmTApDMzNMYVV4ZTZQV1dRUDIwaWRGUHRRclNVRGpMdWN3dTBkbVNLTWZkeGcyenI1eU0ybGdVQjg4UmFoKzF1ZnhFCktqODJCZ09zVW1TelJXSlA5NnBDR0VuMWV0NlNScThhLzZkWXhzSERtTmJLR0tFejR5TldOYkJuYWFJYXJTbGQKNEhjUmFRSnZPZy9qZ0JlWHM4SDVSYlFoQmZ1WXhKNnlVRll2YlJNRDdhQ3ZBY2pNVEsyaU9sSklucmNQM21PNApUdVVtODhHdDlKMzUzN0RqVkx1bnorMGZzeXhlN2p5Mjk1cERIeDVtZ2hnTHJDc041QVhyTzM5bldEbjhYbXRxClNOZHBBZ01CQUFFd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFDQWxJdmZhdHVWa0Y4eStYRVZKYmlRbHd1bGcKZE5ua1VTRVZJNWo5bUhkMzBZdE51Y1J0ZGoyNkZrQldMQmFBb3FIT2ZQSE5wQk1nNitOMXd1T1pTUmc2eElUdApISmt3dlJyNisrS0pKQjJ6b0hSMGMxdXdJWEJYTzBBRFErdE5ZMHY2aStpME5BMWlMenliR2dxUVcwUXE1Y0cyCmNJWE0wUGlNcDBHWVg1ODRuYWcyMHd6VEFTb0w1dlJqTmt2YlBEQUhKRDBsWTdiQlM4SXJRZXRoQ0F4a0FXcisKc3EzQTFjYUJuUjRsa3c2ZTVPTkFUU3dieExVVWlWbTRzKzhaeUtCVXM1SEk3QzM5ZXlpeGpuczllREdMZ0lsUwpqNVU2bUdKTVZyN2NzRTE0QTZEMTBEa0lHY0lYdG0rbEF5eTZBZng0cVdrOUdIQ1g5SmRIdlBEOTdNcz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
      tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2UUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktjd2dnU2pBZ0VBQW9JQkFRQ2tUa0tKTmlYdktMN3UKMVBWbThSOTMyRDdXNHZsOC9hOHNCcTVYSGk1MFE5U3l3ZjBFZnNSdGUwSHU2K2tnaWdjRCtlRTMvVjdnVk9UVApzc2ZMQzMzTGFVeGU2UFdXUVAyMGlkRlB0UXJTVURqTHVjd3UwZG1TS01mZHhnMnpyNXlNMmxnVUI4OFJhaCsxCnVmeEVLajgyQmdPc1VtU3pSV0pQOTZwQ0dFbjFldDZTUnE4YS82ZFl4c0hEbU5iS0dLRXo0eU5XTmJCbmFhSWEKclNsZDRIY1JhUUp2T2cvamdCZVhzOEg1UmJRaEJmdVl4SjZ5VUZZdmJSTUQ3YUN2QWNqTVRLMmlPbEpJbnJjUAozbU80VHVVbTg4R3Q5SjM1MzdEalZMdW56KzBmc3l4ZTdqeTI5NXBESHg1bWdoZ0xyQ3NONUFYck8zOW5XRG44ClhtdHFTTmRwQWdNQkFBRUNnZ0VBZnAvZGdUZGMxc3FWRXlUR0YxYWVoTkwvNHNXN3Rvc2ZsQk4yQ3FlMDcxOVQKTFl4NC9SemhMdXE5N202YkZMdXJHbkphRXJkT2hoNkcxMnZCdEFhZ0pNSjYyKzQzVGx1NTZvZ0g2cURBdlVLYgo4czIyd1NKeXhjUnQrOGxseCtRQUIwRkNmZlZpcks0WDBBcU1rcy9vTlM4L1oyOThNZmk0QXA4QTlMMFpTbmZ4Cm8rbnN5S0NJVFIzSEJ3M2tGWmlrTFNqZkFZSkVlYm00K3p6MkdnRE5nZVM5N0Qrb3ZPbkptczVSMlZBd1pqbnQKQ3QyNW5kZ2ZFY2VkdGEzZ1FkUmk5Rm1NWktJczV3dXFtWkh1bXZ4WkRES0Q5dWQrMXZMS1puMWMxTEpxTXk3NwpxWGdlUzhoeXNIbm9BendrMEtVcUIwWURobElIWUdkUnpqRmVtNTRKblFLQmdRRFFwUjBBTEtGeUh4eDFEQXlHCmhFUGtZbHRUNU1TOXd4cTQrYm5UMldsQ0RHMmZkT3JOMW93OFU3dmp6V1BnMytBS1ZVQUc0YWJOOXJxamt0OFcKMEZMbEMxbytaOXJnSzhpaklYeCtkbVVCcnpEQjJ3VlJTWUZSS2wwS2dQenJZbk92a1djb2JGVFlHWndQM2FaVQp1UEZCazRWNTZBckZYSXlUMnZ2NUpoOXBad0tCZ1FESm1PZVFuam9JblBsWUJWVFJGQ2o2bXhRSVRRRzJXYzhUCnJncGRNNTNvaTcvMVN4Q1J6L1JPRTVhNjFIUWQ0ZXcxclAzZ2tTT0V4bnVRc0s5aGdwSytxd1hTdlMrQkVhZzIKdndpc0NJbUtEZ3hNOTB0cTdRMUdSRDV0aFY0Z0U2NldSOG5mNEtyeWFSd1p6blJrdzZ5SGF5YzFmQkJJdklSOApGQ093R25HbXJ3S0JnSGt4cjZyT1FlazhVUmRjTEZwbXNka1RtT0VlWFhtc3Z2VDdlZ21vbkErVmtJZXpMa0RxCmd3TDMwSWYrWWluWllSWWZkdFdJZFkvbDVYdm1jRmVjSXNxUTBaYTJWTmtxRloxTWNqZ3pKWERaQm9WVVo3NVQKNkIzeGNhSU1VdDJYam9OSS9wYm9kbEFnY0JwM01ZcTg4c2FZbmt1MWthd2FtajI0VWV6alRCTzVBb0dCQUxieQpiZUxOMUhpUWk2OFhWM3ROd2twNmhWbHJHTXkwLzdrcVRmbDZxQ2lxK2c3T2lrRG82Um9acU1Ydm0xaXE5OE5XCk5DYWhVQXhrV3lwWlRTOCtZWkZxZnFSYVQwdmdERGx5YjVvL1BTSHQwYmZmQzdBRFkvS0taK1RZRFMwcTcxc3QKMXNPMmpTdmp1ejZvSHZSNnBvMVY3b1VaQzJZV3Zsd2pvcWRqdUJPOUFvR0FLWk84Vkd5ZXZQRHBNMTRpSVhkRQpqeTAzemhLYkJCR2FRRDk5OCt2REsyNVVvSWJ2cTZtR2p1RUZ0WmFCSUYxZXpUMUsrS1lLbUhQVEtTMDJseWxVCmx0OGIvRkVFVDgvQlM4SFpaWUdnUDZjeGxNYkYxVEZac1RRajJSbmF4WUNzYnN3N0dmTkw4RXl3aldvWE9uOE0KdHRGWFFjSkgvOHQzWUNxWkpGbG1UMjQ9Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K

[Checking `Secret` types here](https://kubernetes.io/docs/concepts/configuration/secret/#secret-types)


Applying the secret manifest

    $ kubectl apply -f secret-tls-nginx-https.yaml

## 2. Creating a `ConfigMap` to pass the https configuration into the nginx instance

### Creating the configuration file `default.conf` 

    server {
      listen 80 default_server;
      listen [::]:80 default_server ipv6only=on;

      listen 443 ssl;

      root /usr/share/nginx/html;
      index index.html;

      server_name localhost;
      ssl_certificate /etc/nginx/ssl/tls.crt;
      ssl_certificate_key /etc/nginx/ssl/tls.key;

      location / {
        try_files $uri $uri/ =404;
      }
    }


### Creating a `ConfigMap` to wrap the `default.conf`

    $ kubectl create configmap configmap-nginx-https-default-conf --from-file=default.conf --namespace project-k 


## 3. Creating the nginx instance with HTTPS enabled 

### Creating the manifest `deployment-nginx-https-loadbalancer.yaml`

    # 
    # the Service object
    #
    apiVersion: v1
    kind: Service
    metadata:
      namespace: project-k
      name: secured-nginx     # the name of the Service object must be the same as the CN for the certificate
      labels:
        app: svc-nginx-https
        namespace: project-k
        env:dev
    spec:
      type: LoadBalancer
      ports:
      - port: 8088
        targetPort: 80
        protocol: TCP
        name: http
      - port: 8443
        protocol: TCP
        name: https
      selector:   # targeting the labels of the pods to be selected
        app: nginx-https
        namespace: project-k
        env:dev

    ---

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      namespace: project-k
      name: deployment-nginx-https
    spec:
      selector:
        matchLabels:
          app: nginx-https
      replicas: 1
      template:
        metadata:
          labels:     # the lables for the Service object to target to the pods
            app: nginx-https      
            namespace: project-k
            env:dev
        spec:
          volumes:
          - name: secret-volume
            secret:
              secretName: secret-tls-nginx-https
          - name: configmap-volume
            configMap:
              name: configmap-nginx-https-default-conf 
          containers:
          - name: container-nginx-https
            image: bprashanth/nginxhttps:1.0
            ports:
            - containerPort: 443
            - containerPort: 80
            volumeMounts:
            - mountPath: /etc/nginx/ssl
              name: secret-volume
            - mountPath: /etc/nginx/conf.d
              name: configmap-volume

### Applying the deployment-nginx-https-loadbalancer.yaml

    $ kubectl apply -f deployment-nginx-https-loadbalancer.yaml

## 4. Accessing from localhost(k8s running on a single node mode)

### Adding entry into `/etc/hosts` file for accessing from localhost

    # add the secured-nginx entry to the /etc/hosts
    127.0.0 secured-nginx

### Accessing the `secured-nginx`

    $ curl --cacert ./ca-self-signed.crt https://secured-nginx:8443

  > the hostname `secured-nginx` 
  > the port `8443`
  > the crt `ca-self-signed.crt`

## 5. Accessing from a pod in the same namespace

### Removing the entry of the `secured-nginx` from `/etc/hosts`

> with the entry set in the `/etc/hosts` file on the single node the cluster running, the internal dns server resolves a wrong record for the `secured-nginx` within the cluster.


### Mounting the TLS `Secret` created into a pod 

    $ kubectl apply -f deployment-curl-with-crt.yaml

  > the self signed certificate file tls.crt locates at /etc/self-signed-ca/ directory of the container. 

### Accessing the `secured-nginx` service from the curl pod in the same namespace

    root@curl# curl --cacert /etc/self-signed-ca/tls.crt https://secured-nginx:8443


## 6. Accessing the `secured-nginx` service from a pod in a different namespace such as `default `

The `secured-nginx` service could be accessed from within the namespaces other than its original namespaces, if the following updates of the deployments is done.
- The certificate should be regenerated with the value of `-subj` change to `/CN=secured-nginx.project-k/O=dev` to match the common name of the certificate with the domain name of the service, because the IP address of the `secured-nginx` service only could be resolved from the DNS name `secured-nginx.project-k` within the namespaces other than `project-k`. 

    $ openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
        -keyout $(pwd)/ca-private-with-namespace.key \
        -out $(pwd)/ca-self-signed-with-namespace.crt \
        -subj "/CN=secured-nginx.project-k/O=dev"


    $ cat ca-private-with-namespace.key | base64 > base64-ca-private-with-namespace.key
    $ cat ca-self-signed-with-namespace.crt | base64 > base64-ca-self-signed-with-namespace.crt

secret-tls-nginx-https.yaml

    apiVersion: v1
    kind: Secret
    metadata: 
      namespace: project-k
      name: secret-tls-nginx-https
    type: kubernetes.io/tls 
    data:
      tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJBQ0NRRGdBV0VNOEE5N216QU5CZ2txaGtpRzl3MEJBUXNGQURBbU1SWXdGQVlEVlFRRERBMXoKWldOMWNtVmtMVzVuYVc1NE1Rd3dDZ1lEVlFRS0RBTmtaWFl3SGhjTk1qSXdOakkyTVRReE1qUXpXaGNOTWpNdwpOakkyTVRReE1qUXpXakFtTVJZd0ZBWURWUVFEREExelpXTjFjbVZrTFc1bmFXNTRNUXd3Q2dZRFZRUUtEQU5rClpYWXdnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFDa1RrS0pOaVh2S0w3dTFQVm0KOFI5MzJEN1c0dmw4L2E4c0JxNVhIaTUwUTlTeXdmMEVmc1J0ZTBIdTYra2dpZ2NEK2VFMy9WN2dWT1RUc3NmTApDMzNMYVV4ZTZQV1dRUDIwaWRGUHRRclNVRGpMdWN3dTBkbVNLTWZkeGcyenI1eU0ybGdVQjg4UmFoKzF1ZnhFCktqODJCZ09zVW1TelJXSlA5NnBDR0VuMWV0NlNScThhLzZkWXhzSERtTmJLR0tFejR5TldOYkJuYWFJYXJTbGQKNEhjUmFRSnZPZy9qZ0JlWHM4SDVSYlFoQmZ1WXhKNnlVRll2YlJNRDdhQ3ZBY2pNVEsyaU9sSklucmNQM21PNApUdVVtODhHdDlKMzUzN0RqVkx1bnorMGZzeXhlN2p5Mjk1cERIeDVtZ2hnTHJDc041QVhyTzM5bldEbjhYbXRxClNOZHBBZ01CQUFFd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFDQWxJdmZhdHVWa0Y4eStYRVZKYmlRbHd1bGcKZE5ua1VTRVZJNWo5bUhkMzBZdE51Y1J0ZGoyNkZrQldMQmFBb3FIT2ZQSE5wQk1nNitOMXd1T1pTUmc2eElUdApISmt3dlJyNisrS0pKQjJ6b0hSMGMxdXdJWEJYTzBBRFErdE5ZMHY2aStpME5BMWlMenliR2dxUVcwUXE1Y0cyCmNJWE0wUGlNcDBHWVg1ODRuYWcyMHd6VEFTb0w1dlJqTmt2YlBEQUhKRDBsWTdiQlM4SXJRZXRoQ0F4a0FXcisKc3EzQTFjYUJuUjRsa3c2ZTVPTkFUU3dieExVVWlWbTRzKzhaeUtCVXM1SEk3QzM5ZXlpeGpuczllREdMZ0lsUwpqNVU2bUdKTVZyN2NzRTE0QTZEMTBEa0lHY0lYdG0rbEF5eTZBZng0cVdrOUdIQ1g5SmRIdlBEOTdNcz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
      tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2UUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktjd2dnU2pBZ0VBQW9JQkFRQ2tUa0tKTmlYdktMN3UKMVBWbThSOTMyRDdXNHZsOC9hOHNCcTVYSGk1MFE5U3l3ZjBFZnNSdGUwSHU2K2tnaWdjRCtlRTMvVjdnVk9UVApzc2ZMQzMzTGFVeGU2UFdXUVAyMGlkRlB0UXJTVURqTHVjd3UwZG1TS01mZHhnMnpyNXlNMmxnVUI4OFJhaCsxCnVmeEVLajgyQmdPc1VtU3pSV0pQOTZwQ0dFbjFldDZTUnE4YS82ZFl4c0hEbU5iS0dLRXo0eU5XTmJCbmFhSWEKclNsZDRIY1JhUUp2T2cvamdCZVhzOEg1UmJRaEJmdVl4SjZ5VUZZdmJSTUQ3YUN2QWNqTVRLMmlPbEpJbnJjUAozbU80VHVVbTg4R3Q5SjM1MzdEalZMdW56KzBmc3l4ZTdqeTI5NXBESHg1bWdoZ0xyQ3NONUFYck8zOW5XRG44ClhtdHFTTmRwQWdNQkFBRUNnZ0VBZnAvZGdUZGMxc3FWRXlUR0YxYWVoTkwvNHNXN3Rvc2ZsQk4yQ3FlMDcxOVQKTFl4NC9SemhMdXE5N202YkZMdXJHbkphRXJkT2hoNkcxMnZCdEFhZ0pNSjYyKzQzVGx1NTZvZ0g2cURBdlVLYgo4czIyd1NKeXhjUnQrOGxseCtRQUIwRkNmZlZpcks0WDBBcU1rcy9vTlM4L1oyOThNZmk0QXA4QTlMMFpTbmZ4Cm8rbnN5S0NJVFIzSEJ3M2tGWmlrTFNqZkFZSkVlYm00K3p6MkdnRE5nZVM5N0Qrb3ZPbkptczVSMlZBd1pqbnQKQ3QyNW5kZ2ZFY2VkdGEzZ1FkUmk5Rm1NWktJczV3dXFtWkh1bXZ4WkRES0Q5dWQrMXZMS1puMWMxTEpxTXk3NwpxWGdlUzhoeXNIbm9BendrMEtVcUIwWURobElIWUdkUnpqRmVtNTRKblFLQmdRRFFwUjBBTEtGeUh4eDFEQXlHCmhFUGtZbHRUNU1TOXd4cTQrYm5UMldsQ0RHMmZkT3JOMW93OFU3dmp6V1BnMytBS1ZVQUc0YWJOOXJxamt0OFcKMEZMbEMxbytaOXJnSzhpaklYeCtkbVVCcnpEQjJ3VlJTWUZSS2wwS2dQenJZbk92a1djb2JGVFlHWndQM2FaVQp1UEZCazRWNTZBckZYSXlUMnZ2NUpoOXBad0tCZ1FESm1PZVFuam9JblBsWUJWVFJGQ2o2bXhRSVRRRzJXYzhUCnJncGRNNTNvaTcvMVN4Q1J6L1JPRTVhNjFIUWQ0ZXcxclAzZ2tTT0V4bnVRc0s5aGdwSytxd1hTdlMrQkVhZzIKdndpc0NJbUtEZ3hNOTB0cTdRMUdSRDV0aFY0Z0U2NldSOG5mNEtyeWFSd1p6blJrdzZ5SGF5YzFmQkJJdklSOApGQ093R25HbXJ3S0JnSGt4cjZyT1FlazhVUmRjTEZwbXNka1RtT0VlWFhtc3Z2VDdlZ21vbkErVmtJZXpMa0RxCmd3TDMwSWYrWWluWllSWWZkdFdJZFkvbDVYdm1jRmVjSXNxUTBaYTJWTmtxRloxTWNqZ3pKWERaQm9WVVo3NVQKNkIzeGNhSU1VdDJYam9OSS9wYm9kbEFnY0JwM01ZcTg4c2FZbmt1MWthd2FtajI0VWV6alRCTzVBb0dCQUxieQpiZUxOMUhpUWk2OFhWM3ROd2twNmhWbHJHTXkwLzdrcVRmbDZxQ2lxK2c3T2lrRG82Um9acU1Ydm0xaXE5OE5XCk5DYWhVQXhrV3lwWlRTOCtZWkZxZnFSYVQwdmdERGx5YjVvL1BTSHQwYmZmQzdBRFkvS0taK1RZRFMwcTcxc3QKMXNPMmpTdmp1ejZvSHZSNnBvMVY3b1VaQzJZV3Zsd2pvcWRqdUJPOUFvR0FLWk84Vkd5ZXZQRHBNMTRpSVhkRQpqeTAzemhLYkJCR2FRRDk5OCt2REsyNVVvSWJ2cTZtR2p1RUZ0WmFCSUYxZXpUMUsrS1lLbUhQVEtTMDJseWxVCmx0OGIvRkVFVDgvQlM4SFpaWUdnUDZjeGxNYkYxVEZac1RRajJSbmF4WUNzYnN3N0dmTkw4RXl3aldvWE9uOE0KdHRGWFFjSkgvOHQzWUNxWkpGbG1UMjQ9Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K


- Redeploy the `Deployment` of the `secured-nginx` service to activate the `secret`.

    $ kubectl scale deployment deployment-nginx-https --replicas=0
    $ kubectl scale deployment deployment-nginx-https --replicas=1
    

- Redeploy the `Pod` curl within the `project-k` namespace, then access the `secured-nginx` service with the `secured-nginx.project-k` domain name from within the namespace `project-k`.

    $ kubectl scale deployment curl --replicas=0
    $ kubectl scale deployment curl --replicas=1
    


- Copy the `secret` to any other namespace need to access the `secured-nginx` service, mount it to the pods which is going to access the `secured-nginx` service.

- Create a `Pod` in the namespace `default` with the secret mounted, to access the `secured-nginx` service.
 
