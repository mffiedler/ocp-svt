# OCP 4.0 on libvirt on GCP
[Dan Mace's documentation](https://github.com/ironcladlou/openshift4-libvirt-gcp)

## Get a pull secret from CoreOS
1. https://account.coreos.com/
1. Register for a 10 node Tectonic license
1. Get the pull secret and store it in a file
## Boot an instance and ssh to it
**Note:** dnf install google-cloud-sdk for the gcloud command
The instance uses the openshift4-libvirt image and the pull secret from above.  Adjust the instance size as desired.
```sh
[mifiedle@mifiedle-rdu-redhat-com ~]$ gcloud compute instances create $INSTANCE   --image-family openshift4-libvirt   --zone us-east1-c   --min-cpu-platform "Intel Haswell"   --machine-type n1-standard-16   --boot-disk-type pd-ssd --boot-disk-size 256GB   --metadata-from-file openshift-pull-secret=/home/mifiedle/work/openshift-pull-secret.json
Created [https://www.googleapis.com/compute/v1/projects/openshift-gce-devel/zones/us-east1-c/instances/mffiedler-test].
NAME            ZONE        MACHINE_TYPE    PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
mffiedler-test  us-east1-c  n1-standard-16               10.240.0.21  35.237.108.60  RUNNING

[mifiedle@mifiedle-rdu-redhat-com ~]$ gcloud compute --project "openshift-gce-devel" ssh --zone "us-east1-c" "mffiedler-test" --ssh-key-file=/home/mifiedle/.ssh/gcloud.pem
Warning: Permanently added 'compute.3344720975295689658' (ECDSA) to the list of known hosts.
[mifiedle@mffiedler-test ~]$ 
```

## Create a single node cluster
```sh
[mifiedle@mifiedle-rdu-redhat-com ~]$ create-cluster nested
```

## Use oc normally to interact with the cluster
```sh
[mifiedle@mffiedler-test ~]$ oc get pods --all-namespaces
NAMESPACE                                NAME                                                              READY     STATUS      RESTARTS   AGE
kube-system                              etcd-member-nested-master-0                                       1/1       Running     0          18m
kube-system                              kube-flannel-gthdz                                                2/2       Running     0          17m
kube-system                              kube-proxy-bgjqw                                                  1/1       Running     0          19m
kube-system                              kube-scheduler-gczjc                                              1/1       Running     0          19m
kube-system                              metrics-server-5767bfc576-6ddfr                                   0/2       Pending     0          14m
kube-system                              tectonic-network-operator-5dlp2                                   1/1       Running     0          19m
openshift-apiserver                      apiserver-d6n6f                                                   1/1       Running     0          14m
openshift-cluster-api                    clusterapi-manager-controllers-6647c9ccd6-22llq                   3/3       Running     0          15m
openshift-cluster-api                    machine-api-operator-59c94f8c74-vbqk6                             1/1       Running     0          19m
openshift-cluster-dns-operator           cluster-dns-operator-58b6cbbb6d-ts4dd                             1/1       Running     0          19m
openshift-cluster-dns                    dns-default-76nht                                                 1/1       Running     0          16m
openshift-cluster-network-operator       cluster-network-operator-68bkj                                    1/1       Running     6          19m
openshift-cluster-node-tuning-operator   cluster-node-tuning-operator-6fb5f8c5dd-4xsw9                     0/1       Pending     0          12m
openshift-cluster-samples-operator       cluster-samples-operator-7b77d8fc58-fcql8                         1/1       Running     0          12m
openshift-cluster-version                cluster-version-operator-vjmfk                                    1/1       Running     0          19m
openshift-console                        console-operator-66bb6d4f7c-h7hd4                                 0/1       Pending     0          12m
openshift-controller-manager             controller-manager-q8kpt                                          1/1       Running     0          16m
openshift-core-operators                 openshift-cluster-kube-apiserver-operator-9f47d9486-hxk24         1/1       Running     0          19m
openshift-core-operators                 openshift-cluster-kube-controller-manager-operator-8654566lk776   1/1       Running     0          19m
openshift-core-operators                 openshift-cluster-kube-scheduler-operator-96ccd6cd7-svqlj         1/1       Running     0          19m
openshift-core-operators                 openshift-cluster-openshift-apiserver-operator-5bb74fb9cd-gjfl4   1/1       Running     0          19m
openshift-core-operators                 openshift-cluster-openshift-controller-manager-operator-655tdb9   1/1       Running     0          19m
openshift-core-operators                 openshift-service-cert-signer-operator-55b84746bf-bh75p           1/1       Running     0          19m
openshift-csi-operator                   csi-operator-798d9957c7-4gngd                                     0/1       Pending     0          12m
openshift-image-registry                 cluster-image-registry-operator-69d8dcc695-scfvz                  1/1       Running     0          12m
openshift-ingress-operator               ingress-operator-588d67ccbb-tg4l4                                 0/1       Pending     0          12m
openshift-kube-apiserver                 installer-1-nested-master-0                                       0/1       Completed   0          15m
openshift-kube-apiserver                 openshift-kube-apiserver-nested-master-0                          1/1       Running     0          15m
openshift-kube-controller-manager        installer-1-nested-master-0                                       0/1       Completed   0          15m
openshift-kube-controller-manager        openshift-kube-controller-manager-nested-master-0                 1/1       Running     0          15m
openshift-kube-scheduler                 scheduler-5dc56f47cc-gbz5n                                        1/1       Running     0          16m
openshift-machine-config-operator        machine-config-operator-588468c4c6-tc8cj                          1/1       Running     0          19m
openshift-monitoring                     cluster-monitoring-operator-67d79b5f88-fql95                      0/1       Pending     0          12m
openshift-operator-lifecycle-manager     catalog-operator-66f47f4c59-mpx47                                 0/1       Pending     0          19m
openshift-operator-lifecycle-manager     olm-operator-5645dd8cd9-vnlnk                                     0/1       Pending     0          19m
openshift-operator-lifecycle-manager     package-server-6d9558f659-xsfpl                                   0/1       Pending     0          19m
openshift-service-cert-signer            apiservice-cabundle-injector-688f959f6d-k4hbn                     1/1       Running     0          16m
openshift-service-cert-signer            configmap-cabundle-injector-769dd649cf-ddnph                      1/1       Running     0          16m
openshift-service-cert-signer            service-serving-cert-signer-54fb45445b-mq4qt                      1/1       Running     0          16m
tectonic-system                          kube-addon-operator-654588755f-mbbbx                              1/1       Running     0          16m
```

