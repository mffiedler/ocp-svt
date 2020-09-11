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

# Creating a jump node in cluster VPC from console (GCP)
## By To Hung Sze - Sept 2020
## Create a VM instsance
1 
export IMAGE_FAMILY="projects/centos-cloud/global/images/family/centos-7"

2
gcloud compute instances create tsze-vpc-bastion1 --image-family=${IMAGE_FAMILY} --subnet=<cluster_infra_id>-master-subnet --tags=<your_tag> --zone=us-central1-b

(<cluster_infra_id>: e.g. cmead-kata17-tkkqb 
zone to match your cluster’s location)

(if you delete the machine and start over again, you can’t use the same name - dns entry will not be updated in time)

## Create Firewall rule
From GCP console, create a firewall and specify <your_infra_id>-network for your cluster as the network, specify the tag from above and allow ssh port 22 and icmp (tag e.g. tsze-vpc-bastion, priority 999, ip range 0.0.0.0/0) 
Test by pinging the host and ssh into the bastion host from GCP console (use browser ssh or ssh terminal - see below) (if you don’t ping, you don’t need to allow icmp)


## ssh into the machine
Try (in order of preference)
1 using openshift-qe.pem
	ssh -i <path-to>/openshfit-qe.pem cloud-user@<ip>

(openshfit-qe key pair in:
https://code.engineering.redhat.com/gerrit/#/admin/projects/cucushift-internal)

2 via gcloud
commandline
	gcloud beta compute ssh --zone “<your_zone>” “<machine_name>” --project “openshift-qe”
Or 	
from gcp console using browser, there is a drop down labeled ssh, choose “Open in browser window”

3 If #1 & #2 don’t work, try 
	ssh -i <path-to>/openshfit-qe.pem <your_redhat_id>@<ip>
Or
	ssh -i <path-to>/openshfit-qe.pem cloud-user@<ip>

(#3 here works if you have added your key to openshift-qe service account when you tried to download openshift-qe key pair which is available in cucushift/cucushift-internal/config/keys repo)

4 If you can ssh in using #3 but wants to use openshift-qe.pem
After ssh in via gcp (commandline or browser)
Create authorized_keys file
pwd to find out the account name for you

ssh core@10.0.0.4 (does not matter if the ip is an actual master that exists, you should not be able to ssh into the machine but should get permission denied and .ssh directory should be created)

Create authorized_keys file under .ssh directory created above
Copy openshift-qe.pub content into the authorized_keys file

chmod 755 .ssh
chmod 644 .ssh/authorized_keys

Note: When you are done and want to destroy the cluster, delete the VM and firewall first - just stopping the VM isn't enough

Reference:
https://docs.google.com/document/d/1kHgPdplX6R8IWW4kia1mhKHjse6eRjBt2-W5ANzFP8o

