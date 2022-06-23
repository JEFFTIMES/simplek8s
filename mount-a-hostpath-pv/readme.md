# Checking the available StorageClasses

    $ kubectl get storageclasses.storage.k8s.io
    NAME                 PROVISIONER          RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
    hostpath (default)   docker.io/hostpath   Delete          Immediate           false                  11d

# Creating a PersistentVolume object and the corresponding PersistentVolumeClaim object

* Crafting the **pv-hostpath-tmp-folder.yaml** file.

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: pv-hostpath-tmp-folder
      labels:
        type: hostPath
        location: host-tmp
    spec:
      storageClassName: host-path-storage-class
      capacity:
        storage: 256Mi
      accessModes:
        - ReadWriteMany
      hostPath:
        path: /tmp
      persistentVolumeReclaimPolicy: Retain
    ---
    #
    # Define a PersistentVolumeClaim to correspond the precedent defined PV 
    #
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: pvc-hostpath-tmp-folder
    spec:
      storageClassName: host-path-storage-class
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 32Mi

* Applying the yaml file.

    $ kubectl apply -f pv-hostpath-tmp-folder.yaml


# Mounting the PersistentVolume to a container


* crafting **curl-nount-pv-by-pvc.yaml** file.
* specifying the `volumes` property in the `spec` property of the `PodTemplate`.
* and the `VolumeMounts` property of the `containers` property.

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: curl-mount-pv-by-pvc
    spec:
      selector:
        matchLabels:
          app: curlpod
      replicas: 1
      template:
        metadata:
          labels:
            app: curlpod
        spec:
          volumes:
          - name: host-tmp-folder
            persistentVolumeClaim: 
              claimName: pvc-hostpath-tmp-folder
          containers:
          - name: curl-with-pv-and-secret
            command:
            - sh
            - -c
            - while true; do sleep 1; done
            image: radial/busyboxplus:curl
            volumeMounts:
            - mountPath: /tmp/thisVol
              subPath: this_folder
              name: host-tmp-folder
            - mountPath: /tmp/thatVol
              subPath: that_folder
              name: host-tmp-folder

# Creating the pod.

    kubectl apply -f curl-mount-pv-by-pvc.yaml