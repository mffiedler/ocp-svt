# Simple SSH bastion for OCP 4
Since public addresses are removed from the master, there are two ways to create a jump node.
1. Boot a node that is part of the VPC (includes oc client).   Script is [here](https://github.com/hongkailiu/svt-case-doc/blob/master/scripts/my_installer_post.sh)
1. eparis ssh bastion
```sh
[xxx@arctic ocp]$ curl https://raw.githubusercontent.com/eparis/ssh-bastion/master/deploy/deploy.sh | bash
...
namespace/openshift-ssh-bastion created
service/ssh-bastion created
secret/ssh-host-keys created
serviceaccount/ssh-bastion created
role.rbac.authorization.k8s.io/ssh-bastion created
rolebinding.rbac.authorization.k8s.io/ssh-bastion created
clusterrole.rbac.authorization.k8s.io/ssh-bastion created
clusterrolebinding.rbac.authorization.k8s.io/ssh-bastion created
deployment.extensions/ssh-bastion created
The bastion address is ad8ce0f66366511e997c306022c6f037-1496953385.us-east-2.elb.amazonaws.com
You may want to use https://raw.githubusercontent.com/eparis/ssh-bastion/master/ssh.sh to easily ssh through the bastion to specific nodes.

[xxx@arctic ocp]$ wget https://raw.githubusercontent.com/eparis/ssh-bastion/master/ssh.sh

Wait couple of minutes then:

[xxx@arctic ocp]$ ./ssh.sh ip-10-0-149-128.us-east-2.compute.internal
Warning: Permanently added 'ad8ce0f66366511e997c306022c6f037-1496953385.us-east-2.elb.amazonaws.com,18.216.25.248' (ECDSA) to the list of known hosts.
Warning: Permanently added 'ip-10-0-149-128.us-east-2.compute.internal' (ECDSA) to the list of known hosts.
Red Hat CoreOS 4.0 Beta
WARNING: Direct SSH access to machines is not recommended.
In a future version of Red Hat CoreOS, this will cause nodes to become tainted.

---
[core@ip-10-0-149-128 ~]$
```