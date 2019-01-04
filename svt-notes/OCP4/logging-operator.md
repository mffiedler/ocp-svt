# OCP 4.0 Logging stack notes
## Install a single node ES cluster with emptydir storage

See also: [Dev instructions](https://github.com/openshift/cluster-logging-operator) for setting up a dev env.

1. git clone https://github.com/openshift/cluster-logging-operator
1. git clone https://github.com/openshift/elasticsearch-operator
1. configure KUBECONFIG (/path/to/installer/auth/kubeconfig)
1. Login as kubeadmin as instructed by installer (pw is in /path/to/installer/auth/kubeadmin-password)
1. cd cluster-logging-operator
1. REMOTE_REGISTRY=true make deploy-example

```sh
$ oc get elasticsearch
NAME            AGE
elasticsearch   42m
$ 
$ oc get pods
NAME                                                  READY     STATUS    RESTARTS   AGE
cluster-logging-operator-5b8f47b598-k6hfl             1/1       Running   0          43m
elasticsearch-clientdatamaster-0-1-84d764899d-26lhr   1/1       Running   2          42m
elasticsearch-operator-649f9b69b5-mjwhd               1/1       Running   0          42m
fluentd-49nhh                                         1/1       Running   0          42m
fluentd-5bd6r                                         1/1       Running   0          42m
fluentd-7f2nj                                         1/1       Running   0          42m
fluentd-86jdb                                         1/1       Running   0          42m
fluentd-np7p4                                         1/1       Running   0          42m
fluentd-vl9x2                                         1/1       Running   0          42m
kibana-675b587dfd-g96cd                               2/2       Running   0          42m
$ 
$ oc get svc
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
cluster-logging-operator   ClusterIP   172.30.168.46    <none>        60000/TCP   43m
elasticsearch              ClusterIP   172.30.32.185    <none>        9200/TCP    43m
elasticsearch-cluster      ClusterIP   172.30.39.229    <none>        9300/TCP    43m
elasticsearch-operator     ClusterIP   172.30.221.126   <none>        60000/TCP   43m
kibana                     ClusterIP   172.30.29.50     <none>        443/TCP     43m
$ 
$ oc get configmap
NAME             DATA      AGE
curator          3         51m
elasticsearch    3         51m
fluentd          3         51m
sharing-config   2         51m
```
**NOTE**:  New service name for ES.   logging-es -> elasticsearch.   All test curl commands need to be modified.

## Change ES cluster size, shards and replicas
- oc scale --replicas=0 deployment/elasticsearch-operator
- oc edit elasticsearch elasticsearch and modify .spec.nodes.replicas
```yaml
spec:
  dataReplication: NoReplication
  managementState: Managed
  nodeSpec:
    image: docker.io/openshift/origin-logging-elasticsearch5:latest
    resources: {}
  nodes:
  - nodeSpec:
      resources: {}
    replicas: 1
    roles:
    - client
    - data
    - master
    storage:
      emptyDir: {}
```
- oc edit configmap elasticsearch and modify index_settings
```yaml
  index_settings: |2

    PRIMARY_SHARDS=1
    REPLICA_SHARDS=0
```
1. oc scale --replicas=1 deployment/elasticsearch-operator

**TODO**:  Real storage config for multi-node ES cluster.  See:  https://github.com/openshift/elasticsearch-operator/pull/55

## Remove logging
1. cd openshift-cluster-operator
1. hack/undeploy.sh