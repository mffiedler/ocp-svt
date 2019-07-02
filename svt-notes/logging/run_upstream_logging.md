# Run 4.x upstream logging with no OperatorHub

This assumes you have a golang dev env with $GOPATH set correctly.   In my example it is /root/go.   Also assumes you have KUBECONFIG exported.
* cd $GOPATH/github.com/openshift (create any missing pieces of that)
* git clone https://github.com/openshift/origin-aggregated-logging 
* cd $GOPATH/github.com/openshift/origin-aggregated-logging
* USE_CUSTOM_IMAGES=true MASTER_VERSION=4.2 EXT_REG_IMAGE_NS=origin EXTERNAL_REGISTRY=registry.svc.ci.openshift.org hack/deploy-logging.sh   

``` bash
oc get pods
NAME                                            READY   STATUS    RESTARTS   AGE
cluster-logging-operator-b445fcc76-qrcxp        1/1     Running   0          6m32s
elasticsearch-cdm-qmapppz3-1-7b7947fd94-jrlvk   2/2     Running   0          6m10s
elasticsearch-operator-687b568fbf-qt9hg         1/1     Running   0          6m30s
fluentd-82m4x                                   1/1     Running   0          6m9s
fluentd-bz5g2                                   1/1     Running   0          6m9s
fluentd-htzw7                                   1/1     Running   0          6m9s
fluentd-jzkgl                                   1/1     Running   0          6m9s
fluentd-wb6dd                                   1/1     Running   0          6m9s
fluentd-x6fdh                                   1/1     Running   0          6m9s
kibana-f746bbdd7-rnmsh                          2/2     Running   0          6m9s
```

oc describe pods should show all pods using images from registry.svc.ci.openshift.org

To uninstall:  hack/undeploy-logging.sh