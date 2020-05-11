# Configuring the kubelet max pod log size
By default the kubelet wraps the current pod log (stdout/stderr) at 10M.   The current and previous log are available uncompressed and older logs are gzip-ed.   To configure larger pod log "keep" sizes, follow the procedure below.

(this is an example, you can choose different label names)

``` bash
oc get machineconfigpool
NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED
master   rendered-master-c0e2b7477278d3349cfc8a2d6edac275   True      False      False
worker   rendered-worker-7e192bb2fb5461fe9acd23860852fb1d   True      False      False
```

Assuming it is the worker nodes where you want to configure larger pod logs:

``` bash
oc label machineconfigpool worker custom-kubelet=bigger-logs        
machineconfigpool.machineconfiguration.openshift.io/worker labeled
```

Create a yaml file with a KubeletConfig resource with a containerLogMaxSize parameter.  The matchLabel should match the label above.  Example:
``` yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: KubeletConfig
metadata:
  name: set-max-log-size 
spec:
  machineConfigPoolSelector:
    matchLabels:
      custom-kubelet: bigger-logs
  kubeletConfig:
    containerLogMaxSize: 200Mi
    containerLogMaxFiles: 10
```

Then create the KubeletConfig, wait a bit and then check the machineconfigpools:

``` bash
oc create -f bigger-logs-node-config.yaml 
kubeletconfig.machineconfiguration.openshift.io/set-max-pods created

oc get machineconfigpools           
NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED
master   rendered-master-c0e2b7477278d3349cfc8a2d6edac275   True      False      False
worker   rendered-worker-46807531cfcc345c2856dcd4cd6d8b0f   False     True       False
```

This will take a while.  Each node will have its scheduling disabled and will be rebooted to apply the change.   Once UPDATED is True again, the change is applied.
