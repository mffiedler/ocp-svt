# OCP 4.0 machine-api
## Configuration
### machineconfigpool

```sh
$ oc get machineconfigpool
NAME      AGE
master    1d
worker    1d
```

Uses label selector to decide which machines are in which pool
```yaml
spec:
  machineConfigSelector:
    matchLabels:
      machineconfiguration.openshift.io/role: worker
  machineSelector:
    matchLabels:
      node-role.kubernetes.io/worker: ""
```
### machineconfig
Seems to be a replacement for the node config map, at least for kubelet flags:
```yaml
    systemd:
      units:
      - contents: |
          [Unit]
          Description=Kubernetes Kubelet
          Wants=rpc-statd.service

          [Service]
          Type=notify
          ExecStartPre=/bin/mkdir --parents /etc/kubernetes/manifests
          EnvironmentFile=-/etc/kubernetes/kubelet-workaround
          EnvironmentFile=-/etc/kubernetes/kubelet-env

          ExecStart=/usr/bin/hyperkube \
              kubelet \
                --config=/etc/kubernetes/kubelet.conf \
                --bootstrap-kubeconfig=/etc/kubernetes/kubeconfig \
                --kubeconfig=/var/lib/kubelet/kubeconfig \
                --container-runtime=remote \
                --container-runtime-endpoint=/var/run/crio/crio.sock \
                --allow-privileged \
                --node-labels=node-role.kubernetes.io/worker,foo=bar \
                --minimum-container-ttl-duration=6m0s \
                --client-ca-file=/etc/kubernetes/ca.crt \
                --cloud-provider=aws \
                \
                --anonymous-auth=false \

          Restart=always
          RestartSec=10
```
Question:  Does changing the machineconfig dynamically update/restart the kubelets the config applies to?

## Relation ship of machines to nodes and instances
Machines are named the same as corresponding AWS instances.   There is 1 machine per node:
```sh
[fedora@ip-172-31-15-221 ~]$ oc get nodes
NAME                           STATUS     ROLES     AGE       VERSION
ip-10-0-134-143.ec2.internal   Ready      worker    1d        v1.11.0+d4cacc0
ip-10-0-147-46.ec2.internal    Ready      worker    1d        v1.11.0+d4cacc0
ip-10-0-166-103.ec2.internal   Ready      worker    1d        v1.11.0+d4cacc0
ip-10-0-2-224.ec2.internal     Ready      master    1d        v1.11.0+d4cacc0
ip-10-0-215-171.ec2.internal   NotReady   worker    27m       v1.11.0+d4cacc0
ip-10-0-22-127.ec2.internal    Ready      master    1d        v1.11.0+d4cacc0
ip-10-0-41-60.ec2.internal     Ready      master    1d        v1.11.0+d4cacc0
ip-10-0-5-29.ec2.internal      NotReady   master    18m       v1.11.0+d4cacc0
[fedora@ip-172-31-15-221 ~]$ oc get machines
NAME                               AGE
mifiedle-master-0                  1d
mifiedle-master-1                  1d
mifiedle-master-2                  1d
mifiedle-master-3                  20m
mifiedle-worker-us-east-1a-qrg5h   1d
mifiedle-worker-us-east-1b-5754w   1d
mifiedle-worker-us-east-1c-99ng5   1d
mifiedle-worker-us-east-1f-42pjr   28m
```
## machine sets
Next gen installer creates machinesets for each zone in us-east-1:
```sh
[fedora@ip-172-31-15-221 ~]$ oc get machinesets
NAME                         AGE
mifiedle-worker-us-east-1a   1d
mifiedle-worker-us-east-1b   1d
mifiedle-worker-us-east-1c   1d
mifiedle-worker-us-east-1d   1d
mifiedle-worker-us-east-1e   1d
mifiedle-worker-us-east-1f   1d
```
To scale up/down in a zone, edit the replicas number.   Once saved, instance creation will start followed by the addition of the node.
Question:  How to customize scaled up instance details.   Answer:  edit the machineset:

```yaml
      providerConfig:
        value:
          ami:
            arn: null
            filters: null
            id: ami-0da77f232c6086c9d
          apiVersion: aws.cluster.k8s.io/v1alpha1
          credentialsSecret: null
          deviceIndex: 0
          iamInstanceProfile:
            arn: null
            filters: null
            id: mifiedle-worker-profile
          instanceType: t2.medium
```

Question:  How to scale up master?    I was able to scale up a new master instance and see the node added by the following:
```
oc get -o yaml --export machine mifiedle-master-0 > mifiedle-master.yaml
Edit the file and s/master-0/master-3/g and save
oc create -f mifiedle-master.yaml
```
The new instance and node were created but it got stuck in NotReady so not sure if it was really going to be a master.   More investigation required.