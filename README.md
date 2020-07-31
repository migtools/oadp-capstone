# oadp-capstone

# Step 1: Add OADP Operator to OLM

Clone the OADP Operator repository:
```
git clone git@github.com:konveyor/oadp-operator.git
```

Build the operator image:
```
podman build -f build/Dockerfile . -t oadp-operator:latest
```

Push the image to a registry. For example,
```
podman push localhost/oadp-operator:latest <REGISTRY_URL>
```

In order to use a locally built image of the operator, please update the operator.yaml file. Update the image of the oadp-operator container with the image registry URL. You can edit the file manually or use the following command( <REGISTRY_URL> is the placeholder for your own registry url in the command):
```
sed -i 's|quay.io/konveyor/oadp-operator:latest|<REGISTRY_URL>|g' deploy/operator.yaml
```

Create an `oadp-operator-source.yaml` file like below in oadp-operator directory:
```
apiVersion: operators.coreos.com/v1
kind: OperatorSource
metadata:
  name: oadp-operator
  namespace: openshift-marketplace
spec:
  type: appregistry
  endpoint: https://quay.io/cnr
  registryNamespace: deshah
  displayName: "OADP Operator"
  publisher: "deshah@redhat.com"
```

Remove the deployed resources:
```
oc delete -f deploy/crds/konveyor.openshift.io_v1alpha1_velero_cr.yaml
oc delete -f deploy/crds/konveyor.openshift.io_veleros_crd.yaml   
oc delete -f deploy/
oc delete namespace oadp-operator
oc delete crd $(oc get crds | grep velero.io | awk -F ' ' '{print $1}')
oc delete -f oadp-operator-source.yaml
```

Run the following commands (note: they should be run from the root of the oadp-operator directory):

```
oc create namespace oadp-operator
oc project oadp-operator
oc create secret generic <SECRET_NAME> --namespace oadp-operator --from-file cloud=<CREDENTIALS_FILE_PATH>
oc create -f oadp-operator-source.yaml
```

# Step 2: Install OADP Operator from OperatorHub

Navigate to the OpenShift console in your web browser. Under the Administrator view, go to Operators on the left tab and click on OperatorHub. Search for the OADP Operator in the search bar. Click on it to install and subscribe to the operator. 

![OADP OperatorHub](/images/oadp_operatorhub.png)

If you go to Installed Operators and select the oadp-operator project, you should see the OADP Operator successfully installed:

![OADP Installed](/images/oadp_installed.png)


# Step 3: Install OCS from OperatorHub

Navigate to the OpenShift console. Under the Administrator view, go to Operators on the left tab and click on OperatorHub. Search for the OpenShift Container Storage operator in the search bar. Click on it to install and subscribe to the operator. Make sure to install it under the oadp-operator namespace. 

![OCS OperatorHub](/images/ocs_operatorhub.png)

If you go to Installed Operators and select the openshift-operator project, you should see the OpenShift Container Storage operator successfully installed:

![OCS Installed](/images/ocs_installed.png)

# Step 4: Create a Velero Custom Resource to install Velero+Restic+Noobaa

In order to use OLM for OADP deployment, you need to change flag `olm_managed` in the `konveyor.openshift.io_v1alpha1_velero_cr.yaml` to true. The file is present in deploy/crds folder.

Moreover, to use Nooba, in the `konveyor.openshift.io_v1alpha1_velero_cr.yaml`, you need to:  

1. Set the flag `noobaa` to true.
2. Set the volume snapshot location region to the correct value. 

For instance the `konveyor.openshift.io_v1alpha1_velero_cr.yaml` file might look something like this:

```
apiVersion: konveyor.openshift.io/v1alpha1
kind: Velero
metadata:
  name: example-velero
spec:
  use_upstream_images: true
  olm_managed: true
  noobaa: true
  default_velero_plugins:
  - aws
  backup_storage_locations:
  - name: default
    provider: aws
    object_storage:
      bucket: myBucket
      prefix: "velero"
    config:
      region: us-east-1
      profile: "default"
    credentials_secret_ref:
      name: cloud-credentials
      namespace: oadp-operator
  volume_snapshot_locations:
  - name: default
    provider: aws
    config:
      region: us-west-1
      profile: "default"
  enable_restic: true
```

When the installation succeeds, create a Velero CR
```
oc create -f deploy/crds/konveyor.openshift.io_v1alpha1_velero_cr.yaml
```

Post completion of all the above steps, you can check if the operator was successfully installed; the expected result for the command `oc get all -n oadp-operator` is as follows:
```
NAME                                      READY   STATUS    RESTARTS   AGE
pod/aws-s3-provisioner-6cdf54b89-8nz7l    1/1     Running   0          162m
pod/noobaa-core-0                         1/1     Running   2          160m
pod/noobaa-db-0                           1/1     Running   0          160m
pod/noobaa-endpoint-7f4bcfb665-wmfhf      1/1     Running   0          156m
pod/noobaa-operator-7b79bf7c68-wz5jc      1/1     Running   0          163m
pod/oadp-operator-69fc6bfcb4-xjn2l        1/1     Running   1          161m
pod/ocs-operator-7b564dc46f-hzh8g         1/1     Running   0          163m
pod/rook-ceph-operator-6f985689b4-t95k4   1/1     Running   0          163m

NAME                            TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)                                                    AGE
service/noobaa-db               ClusterIP      172.30.29.6      <none>                                                                    27017/TCP                                                  160m
service/noobaa-mgmt             LoadBalancer   172.30.222.31    a1276fa043dc5469caaa9f6b566b5091-1561678137.us-west-1.elb.amazonaws.com   80:31438/TCP,443:31408/TCP,8445:32375/TCP,8446:31952/TCP   160m
service/oadp-operator-metrics   ClusterIP      172.30.171.159   <none>                                                                    8383/TCP,8686/TCP                                          161m
service/s3                      LoadBalancer   172.30.184.59    aebf6dedc595e489496daaeb74c65f12-400970079.us-west-1.elb.amazonaws.com    80:31321/TCP,443:30213/TCP,8444:32073/TCP                  160m

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/aws-s3-provisioner   1/1     1            1           163m
deployment.apps/noobaa-endpoint      1/1     1            1           156m
deployment.apps/noobaa-operator      1/1     1            1           163m
deployment.apps/oadp-operator        1/1     1            1           161m
deployment.apps/ocs-operator         1/1     1            1           163m
deployment.apps/rook-ceph-operator   1/1     1            1           163m

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/aws-s3-provisioner-5dd8cccfd8   0         0         0       163m
replicaset.apps/aws-s3-provisioner-6cdf54b89    1         1         1       162m
replicaset.apps/noobaa-endpoint-7f4bcfb665      1         1         1       156m
replicaset.apps/noobaa-operator-7b79bf7c68      1         1         1       163m
replicaset.apps/oadp-operator-69fc6bfcb4        1         1         1       161m
replicaset.apps/ocs-operator-7b564dc46f         1         1         1       163m
replicaset.apps/rook-ceph-operator-6f985689b4   1         1         1       163m

NAME                           READY   AGE
statefulset.apps/noobaa-core   1/1     160m
statefulset.apps/noobaa-db     1/1     160m

NAME                                                  REFERENCE                    TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/noobaa-endpoint   Deployment/noobaa-endpoint   0%/80%    1         1         1          156m

NAME                                   HOST/PORT                                                                PATH   SERVICES      PORT         TERMINATION   WILDCARD
route.route.openshift.io/noobaa-mgmt   noobaa-mgmt-oadp-operator.apps.cluster-dlu-4-5.dlu-4-5.mg.dog8code.com          noobaa-mgmt   mgmt-https   reencrypt     None
route.route.openshift.io/s3            s3-oadp-operator.apps.cluster-dlu-4-5.dlu-4-5.mg.dog8code.com                   s3            s3-https     reencrypt     None
```

Also, `oc get noobaa` should give:
```
NAME     MGMT-ENDPOINTS                 S3-ENDPOINTS                   IMAGE                                                                                                            PHASE   AGE
noobaa   [https://10.0.185.183:31408]   [https://10.0.185.183:30213]   registry.redhat.io/ocs4/mcg-core-rhel8@sha256:689c5a109b81190ddc507b17b7b44ae00029951e7e2c80a6e33358a53945dab6   Ready   161m
```


