# Debugging a node NotReady issue with  a root cause of kube-apiserver CrashLooping 

## Get nodes and get clusteroperators
```bash
root@ip-172-31-49-9: ~ # oc get nodes
NAME                                         STATUS     ROLES    AGE     VERSION
ip-10-0-133-15.us-west-2.compute.internal    NotReady   worker   5h54m   v1.12.4+0cbcfc5afe
ip-10-0-138-148.us-west-2.compute.internal   Ready      master   6h13m   v1.12.4+0cbcfc5afe
ip-10-0-156-5.us-west-2.compute.internal     NotReady   worker   5h54m   v1.12.4+0cbcfc5afe
ip-10-0-156-53.us-west-2.compute.internal    Ready      master   6h13m   v1.12.4+0cbcfc5afe
ip-10-0-162-27.us-west-2.compute.internal    Ready      master   6h13m   v1.12.4+0cbcfc5afe
ip-10-0-162-34.us-west-2.compute.internal    NotReady   worker   5h54m   v1.12.4+0cbcfc5afe
root@ip-172-31-49-9: ~ # oc get co
NAME                                  VERSION   AVAILABLE   PROGRESSING   FAILING   SINCE
cluster-autoscaler                              True        False         False     5h58m
cluster-storage-operator                        True        False         False     5h55m
console                                         True        True          True      5h51m
dns                                             True        False         False     6h11m
image-registry                                  True        False         False     5h54m
ingress                                         False       False         False     53m
kube-apiserver                                  True        True          True      3h58m
kube-controller-manager                         True        False         False     3h58m
kube-scheduler                                  True        True          True      3h57m
machine-api                                     True        False         False     6h
machine-config                                  True        False         False     5h59m
marketplace-operator                            True        False         False     5h56m
monitoring                                      False       True          True      45m
network                                         False       True          False     50m
node-tuning                                     False       True          False     50m
openshift-apiserver                             False       False         False     11m
openshift-authentication                        True        False         False     5h54m
openshift-cloud-credential-operator             True        False         False     6h
openshift-controller-manager                    True        False         False     142m
openshift-samples                               True        False         False     5h55m
operator-lifecycle-manager                      True        False         False     5h58m
```
## Look at co kube-apiserver operator - should have errors in it
```bash
root@ip-172-31-49-9: ~ # oc get co/kube-apiserver -o yaml | less                                                                                                                                                                       
apiVersion: config.openshift.io/v1                                                                                                                                                                                                     
kind: ClusterOperator
metadata:
  creationTimestamp: 2019-02-28T15:18:31Z
  generation: 1
  name: kube-apiserver
  resourceVersion: "238158"
  selfLink: /apis/config.openshift.io/v1/clusteroperators/kube-apiserver
  uid: 1baa863a-3b6c-11e9-a2bb-0ab1c8e4c354
spec: {}
status:
  conditions:
  - lastTransitionTime: 2019-02-28T17:10:16Z
    message: "NodeInstallerFailing: 0 nodes are failing on revision 16:\nNodeInstallerFailing:
      \nStaticPodsFailing: nodes/ip-10-0-162-27.us-west-2.compute.internal pods/kube-apiserver-ip-10-0-162-27.us-west-2.compute.internal                                                                                              
      container=\"kube-apiserver-17\" is not ready\nStaticPodsFailing: nodes/ip-10-0-162-27.us-west-2.compute.internal                                                                                                                
      pods/kube-apiserver-ip-10-0-162-27.us-west-2.compute.internal container=\"kube-apiserver-17\"
      is waiting: \"CrashLoopBackOff\" - \"Back-off 2m40s restarting failed container=kube-apiserver-17
      pod=kube-apiserver-ip-10-0-162-27.us-west-2.compute.internal_openshift-kube-apiserver(33662d0dea76fa54d5ec64e39b48861c)\""                                                                                                      
    reason: MultipleConditionsMatching
    status: "True"
    type: Failing
  - lastTransitionTime: 2019-02-28T17:05:50Z
    message: 'Progressing: 3 nodes are at revision 10'
```
## Check kube-apiserver pods
```bash
root@ip-172-31-49-9: ~ # oc get pods -n openshift-kube-apiserver | grep -v Completed
NAME                                                           READY   STATUS             RESTARTS   AGE
installer-13-ip-10-0-162-27.us-west-2.compute.internal         0/1     Error              0          127m
kube-apiserver-ip-10-0-138-148.us-west-2.compute.internal      1/1     Running            0          145m
kube-apiserver-ip-10-0-156-53.us-west-2.compute.internal       1/1     Running            0          146m
kube-apiserver-ip-10-0-162-27.us-west-2.compute.internal       0/1     CrashLoopBackOff   6          7m36s
revision-pruner-8-ip-10-0-162-27.us-west-2.compute.internal    0/1     OOMKilled          0          4h3m
```
## Describe CrashLoopBackoff pod to see snippet of error logs
oc describe pod on a kube-apiserver pod (and I suspect openshift-api-server) and the 2 controller-manager pod types shows the latest error messages from the logs.   Handy if kubelet not reachable for oc adm node-logs or oc debug node
```bash
# oc describe pod kube-apiserver-ip-10-0-162-27.us-west-2.compute.internal -n openshift-kube-apiserver
<snip>
healthz check failed
I0228 19:25:06.312729       1 cache.go:39] Caches are synced for AvailableConditionController controller
I0228 19:25:06.371809       1 controller_utils.go:1034] Caches are synced for crd-autoregister controller
E0228 19:25:06.456573       1 autoregister_controller.go:190] v1.k8s.cni.cncf.io failed with : apiservices.apiregistration.k8s.io "v1.k8s.cni.cncf.io" already exists
E0228 19:25:06.456697       1 autoregister_controller.go:190] v1alpha1.dns.openshift.io failed with : apiservices.apiregistration.k8s.io "v1alpha1.dns.openshift.io" already exists
E0228 19:25:06.456775       1 autoregister_controller.go:190] v1.machineconfiguration.openshift.io failed with : apiservices.apiregistration.k8s.io "v1.machineconfiguration.openshift.io" already exists
W0228 19:25:06.569267       1 lease.go:226] Resetting endpoints for master service "kubernetes" to [10.0.138.148 10.0.156.53 10.0.162.27]
E0228 19:25:06.615295       1 memcache.go:140] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
E0228 19:25:06.909113       1 memcache.go:140] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
F0228 19:25:06.967804       1 hooks.go:188] PostStartHook "crd-discovery-available" failed: unable to retrieve the complete list of server APIs: metrics.k8s.io/v1beta1: the server is currently unable to handle the request
```