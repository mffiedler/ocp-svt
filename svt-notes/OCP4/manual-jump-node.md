# Creating a jump node in cluster VPC from console (AWS)
## Open up port 22 in your security group
1. Find a master instance in the AWS console and note which VPC and security group it is using
1. Go to Security Groups and find that group and highlight it
1. Click the Inbound tab and Edit rules
1. Find the rule for port 22 and change the CIDR to 0.0.0.0/0

## Boot an instance that can join the network
1. In AWS console go to VPC and then Subnets
1. Filter on your cluster name and find the subnet with CIDR 10.0.0.0/16 and node the ID
1. Launch an AMI (Fedora or RHEL)
1. On the Configure Instance tab, choose the VPC and subnet from above and select Auto-assign Public IP enable
1. Choose storage and tags normally
1. On security group step, choose your security group 

When the instance boots, SSH using your key and you should be able to SSH to nodes in your cluster using private IPs.  You can install oc client, kubeconfig, test repos etc as needed.

**IMPORTANT** Terminate this instance before running destroy cluster or else it will hang/timeout and you will have manual cleanup to do.


# Creating a jump node in cluster VPC from console (AZURE)
## Boot an instance that can join the network
1. Click on Virtual Machines on the left pane and then click on +ADD.
1. Under Basics - 
   - Select "Resource group" aligned with your cluster, Provide virtual machine name like "qe-<cluster_name>-jumpnode" and   
     Region would be good to be aligned with your master instance for e.g. (US)Central US
   - For image - click on browse public and private images and chose the private one which better suits your needs for e.g 
     qe- rhel-7-release
   - Administrator account : Select "SSH public key" -> username: redhat-> libra.pub contents in SSH public key
   Leave rest settings as is under Basics
1. Disks type - Standard SSD would suffice
1. Under Networking -
   - Choose Virtual Networks aligned to your exisiting cluster
   - Subnet : Choose your master subnet
   Leave rest settings as is under Networking
1. Under Management
   - Select Diagnostic Storage account as aligned to your cluster name
1. Advanced settings and Tags - Leave as is
1. Click Review and Create and Wait for Deployment to be Complete.

## Open up port 22 in your security group
1. Now click on the Resource Group and then corresponding "Network Security Group" Name. It would be something like 
   qe-xxx-xxxx-xxx-controlplane-nsg
   - Go to the left and click "Inbound Security Rules". We need to add port 22 for SSH access here. Just add Destination 
     port range as 22 and provide a name like Port_22. Leave rest as is.
1. Now click on Virtual Machine son your left and search your jump node. Click on it and you will see its public ip on the
   right pane
1. Now you should be able to go in that jump node as ssh -i libra.pem redhat@<public_ip> and then to any master node from it

**IMPORTANT** Delete this instance before destroying cluster or else it will hang/timeout and you will have manual cleanup to do.
