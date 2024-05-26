
# How to the Deploy GKE clusters from google cloud shell


## Prerequisites

1. Set environment variables
#
    export my_region=us-east1
    export my_cluster=autopilot-cluster-1
## Create a Kubernetes cluster
1. Enter the command below to create a kubernetes cluster. This operation may take up to 10minutes. 

#
    gcloud container clusters create-auto $my_cluster --region $my_region

Additional options can be found in the [documentation.](https://cloud.google.com/sdk/gcloud/reference/container/clusters/create)

## Connect to the GKE cluster from cloudshell

We will will authenticate to the cluster from cloudshell. Authentication in Kubernetes applies both to communicating with the cluster from an external client through the kube-APIserver running on the master and to cluster containers communicating within the cluster or externally.
In Kubernetes, authentication can take several forms. For GKE, authentication is typically handled with OAuth2 tokens and can be managed through Cloud Identity and Access Management across the project as a whole and, optionally, through role-based access control which can be defined and configured within each cluster. In GKE, cluster containers can use service accounts to authenticate to and access external resources. 
client certificates and basic authentication should be disabled by default. These are lower security methods of authentication and should be disabled to increase cluster security. (For versions 1.12 and later, both of these methods are disabled by default.)


We need to create a kubeconfig file with the credentials of the current user (to allow authentication) and provide the endpoint details for a specific cluster (to allow communicating with that cluster through the kubectl command-line tool)
We will execute the following command:

#
    gcloud container clusters get-credentials $my_cluster --region $my_region

This command creates a `.kube` directory in your home directory if it doesn't already exist. In the .kube directory, the command creates a file named `config` if it doesn't already exist, which is used to store the authentication and configuration information. The config file is typically called the kubeconfig file.

To view the content of the kubeconfig file with an editor, run the command below:

#
    nano .kube/config

The information contained in the kubeconfig file was populated during cluster creation.

## Use kubectl to inspect the cluster

After the kubeconfig file is populated and the active context is set to a particular cluster, you can use the kubectl command-line tool to execute commands against the cluster. Most such commands ultimately trigger a REST API call against the master API server, which triggers the associated action.

1. print out the content of the kubeconfig file:

#
    kubectl config view

The sensitive certificate data is replaced with DATA+OMITTED.

2. print out the cluster information for the active context:

#
    kubectl cluster-info

This output describes the active context cluster.

3. print out the active context:
#
    kubectl config current-context

A line of output indicates the active context cluster.

4. print out some details for **all** the cluster contexts in the kubeconfig file:

#
    kubectl config get-contexts

Several lines of output indicate details about the cluster you created and indicate which is the active context cluster. In general, this command lists some details of the clusters present in the user's kubeconfig file, including any other clusters that were created by the user as well as any manually added to the kubeconfig file.

5. to change the active context:

#
    kubectl config use-context gke_${DEVSHELL_PROJECT_ID}_us-east1_autopilot-cluster-1

Since we have only one cluster, the command does not change anything.
However, in the future you may have more than one cluster in a project. You can use this approach to switch the active context when your kubeconfig file has the credentials and configuration for several clusters already populated. This approach requires the full name of the cluster, which includes the gke prefix, the project ID, the location, and the display name, all concatenated with underscores.

6. to enable bash autocompletion for kubectl:

#
    source <(kubectl completion bash)

Although this command produces no output. Try running kubectl followed by a space and co, then press tab key twice. The result will be similar to the image below.

## Deploy pods to GKE cluster
Kubernetes introduces the abstraction of a Pod to group one or more related containers as a single entity to be scheduled and deployed as a unit on the same node. You can deploy a Pod that is a single container from a single container image. Or a Pod can contain many containers from many container images.

1. to deploy nginx as a Pod named nginx-1:

#
    kubectl create deployment --image nginx nginx-1

This command creates a Pod named nginx with a container running the nginx image. When a repository isn't specified, the default behavior is to try to find the image either locally or in the Docker public registry. In this case, the image is pulled from the Docker public registry.

2. to view all the deployed Pods in the active context cluster:

#
    kubectl get pods

3. to view the resource usage across the nodes of the cluster:

#
    kubectl top nodes

4. We will enter our Pod name into a variable that we will use throughout this lab. Using variables like this can help us minimize human error when typing long names. (Use `kubectl get pods` to get the name of pod)

#
    export my-nginx-pod=nginx-1-695d6d476c-blxvj

5.  Confirm that you have set the environment variable successfully 

#
    echo $my_nginx_pod

6. to view the complete details of the Pod you just created:

#
    kubectl describe pod $my_nginx_pod

### Push a file into a container

7. To be able to serve static content through the nginx web server, you must create and place a file into the container.

- open a file named test.html in the nano text editor:

#
    nano ~/test.html

- Add the following text (shell script) to the empty test.html file:

#
    This is title
    Hello world

- place the file into the appropriate location within the nginx container in the nginx Pod to be served statically: 

#
    kubectl cp ~/test.html $my_nginx_pod:/usr/share/nginx/html/test.html

This command copies the test.html file from the local home directory to the `/usr/share/nginx/html` directory of the first container in the nginx Pod. You can specify other containers in a multi-container Pod by using the -c option, followed by the name of the container.

### Expose the Pod for testing
8. 
- We will execute the following command in Cloud Shell,to create a service to expose our nginx Pod externally:

#
    kubectl expose pod $my_nginx_pod --port 80 --type LoadBalancer

This command creates a LoadBalancer service, which allows the nginx Pod to be accessed from internet addresses outside of the cluster.

to view details about services in the cluster:

#
    kubectl get services

9. verify that the nginx container is serving the static HTML file that you copied.
The external IP is displayed when you run `kubectl get svc`
#
    curl http://[EXTERNAL_IP]/test.html

- execute the following command to view the resources being used by the nginx Pod:

#
    kubectl top pods

## Introspect GKE Pods

1. Prepare the environment

The preferred way of deploying Pods and other resources to Kubernetes is through configuration files, which are sometimes called manifest files. Configuration files are typically written in the YAML syntax, specifying the details of the resource. With configuration files, you can more easily specify complex options than with a long line of command-line arguments.

-  clone the repository to the lab Cloud Shell:

#
    git clone https://github.com/GoogleCloudPlatform/training-data-analyst

- Create a soft link as a shortcut to the working directory:

#
    ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s

- Change to the directory that contains the sample files for this lab:

#
    cd ~/ak8s/GKE_Shell/

- To deploy your manifest, execute the following command:
#
    kubectl apply -f ./new-nginx-pod.yaml

- list the pods:
#
    kubectl get pods

- execute the following command to start an interactive bash shell in the nginx container:

#
    kubectl exec -it new-nginx -- /bin/bash

If the Pod had several containers, you could specify one by name with the -c option.

Because the nginx container image has no text editing tools by default, you need to install one.

#
    apt-get update
    apt-get install nano

- execute the following commands to switch to the static files directory and create a test.html file:
#
    cd /usr/share/nginx/html
    nano test.html

- in the nginx bash shell nano session, type the following text and exit the nginx bash shell:
#
    This is title
    Hello world 


- To connect to and test the modified nginx container (with the new static HTML file), you could create a service. An easier way is to use port forwarding to connect to the Pod directly from Cloud Shell.

- to set up port forwarding from Cloud Shell to the nginx Pod (from port 10081 of the Cloud Shell VM to port 80 of the nginx container):

#
    kubectl port-forward new-nginx 10081:80

- execute the following command to test the modified nginx container through the port forwarding, in another cloudshell terminal:

#
    curl http://127.0.0.1:10081/test.html


- execute the following command in another new terminal to display the logs and to stream new logs as they arrive (and also include timestamps) for the new-nginx Pod:

# 
    kubectl logs new-nginx -f --timestamps

- Return to the second Cloud Shell window and re-run the curl command to generate some traffic on the Pod.

- Close the original Cloud Shell window to stop the port forwarding process.
