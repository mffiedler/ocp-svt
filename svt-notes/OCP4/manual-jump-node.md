# Creating a jump node in cluster VPC from console
## Open up port 22 in your security group
1. Find a master instance in the AWS console and note which VPC and security group it is using
1. Go to Security Groups and find that group and highlight it
1. Click the Inbound tab and Edit rules
1. Find the rule for port 22 and change the CIDR to 0.0.0.0

## Boot an instance that can join the network
1. In AWS console go to VPC and then Subnets
1. Filter on your cluster name and find the subnet with CIDR 10.0.0.0/16 and node the ID
1. Launch an AMI (Fedora or RHEL)
1. On the Configure Instance tab, choose the VPC and subnet from above and select Auto-assign Public IP enable
1. Choose storage and tags normally
1. On security group step, choose your security group 

When the instance boots, SSH using your key and you should be able to SSH to nodes in your cluster using private IPs.  You can install oc client, kubeconfig, test repos etc as needed.

**IMPORTANT** Terminate this instance before running destroy cluster or else it will hang/timeout and you will have manual cleanup to do.
