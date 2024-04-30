# How to set up a private Kubernetes Cluster on Google Cloud

Kubernetes Engine (GKE) offers private clusters for enhanced security. These clusters isolate your workloads by keeping the master (control plane) hidden from the public internet. Nodes, the workhorses running your applications, also operate within a private network, using internal IP addresses instead of public ones. This isolation creates a secure environment for your applications.

### Prerequisites
- experience creating and launching Kubernetes Clusters and be 
- CIDR blocks

## Outline
Step 0. Set the region and zone

Step 1. Create private cluster

Step 2. View your subnet and secondary address ranges

Step 3 Enable the master authorized networks

Step 4 Verify 

Step 5. Clean up



## Step 0.  Set the region and zone

Create the environmental variable for the region and zone and config set them.

#
    export REGION=europe-west4;export ZONE=europe-west4-b

#
    gcloud config set compute/region REGION;gcloud config set compute/zone ZONE
## Step 1. Create private cluster
We will create a cluster named `private-cluster` with a CIDR range of 172.16.0.16/28 for the masters(Note that ypou must specify a CIDR range of /28 for the VMS that run the master):

#
    gcloud beta container clusters create private-cluster \
    --enable-private-nodes \
    --master-ipv4-cidr 172.16.0.16/28 \
    --enable-ip-alias \
    --create-subnetwork ""


The cluster will take some time to be created.
##   Step 2. View your subnet and secondary address ranges  

To list the subnet in our network. Run the command below: (We created the cluster in the default network).

#
    gcloud compute networks subnets list --network default 

The output shows the subnet for your cluster. Mine is
`gke-private-cluster-subnet-2176dcd1`

To get the information about your automatically created subnet, run:
#
    gcloud compute networks subnets describe [SUBNET_NAME] --region=$REGION

Replace [SUBNET_NAME] with the subnet for your cluster.

IMAGE
## Step 3 Enable the master authorized networks 
 At this point, the only IP addresses that have access to the master are the addresses in these ranges:

The primary range of your subnetwork. This is the range used for nodes.
The secondary range of your subnetwork that is used for pods.
To provide additional access to the master, you must authorize selected address ranges.

- Create a VM instance named `source-instance`to check the connectivity to Kubernetes clusters:
#
    gcloud compute instances create source-instance --zone=$ZONE --scopes 'https://www.googleapis.com/auth/cloud-platform'


- Get the external IP of the `source-instance` with the following command:
#
    gcloud compute instances describe source-instance --zone=$ZONE | grep natIP

- Run the following to Authorize your external address range, replacing [MY_EXTERNAL_RANGE] with the CIDR range of the external addresses from the previous output (your CIDR range is natIP/32). With CIDR range as natIP/32, we are allowlisting one specific IP address:

#
    gcloud container clusters update private-cluster \
    --enable-master-authorized-networks \
    --master-authorized-networks [MY_EXTERNAL_RANGE]
## Step 4 Verify   
Now that you have access to the master from a range of external addresses, you'll install kubectl so you can use it to get information about your cluster. For example, you can use kubectl to verify that your nodes do not have external IP addresses.

1. SSH into source-instance:
#
    gcloud compute ssh source-instance --zone=$ZONE

2. Install kubectl in the ssh shell:
#
   sudo apt-get install kubectl 

3. Create the environment variable of the zone and Configure access to the Kubernetes cluster from SSH shell with:

#
    export ZONE=europe-west4-b
#
    sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin gcloud container clusters get-credentials private-cluster --zone=$ZONE

4. Verify that the cluster nodes do not have external IP addresses:
#
    kubectl get nodes --output yaml | grep -A4 addresses

or with the following command:

#
    kubectl get nodes --output wide

    
## Step 5. Clean up
Delete the cluster by running the following command
#
    gcloud container clusters delete private-cluster --zone=$ZONE