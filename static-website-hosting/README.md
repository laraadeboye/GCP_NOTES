# How to host a static content on a google cloud storage bucket and a google cloud compute engine


## Outline
- Prerequisites
- Hosting a static content on a google cloud storage bucket
- Hosting a static website on a google cloud compute engine.
- Conclusion



## Prerequisites

1. Previously created project in a Google cloud account

2. Activate cloudshell in the google cloud console.

3. We will create environment variables for the project.
#
    MY_BUCKET_NAME=sample-bucket-234
    REGION=us-central1
    ZONE=us-central1-cMY_VM=webvm

4. Set the default zone and region for the project.

#
    gcloud config set compute/zone $ZONE

#
    gcloud config set compute/zone $REGION



## Hosting a static content on a google cloud storage bucket

1. Create google cloud storage. We can use either the gcloud storage or gsutil to create our bucket. We will be using the gcloud storage command

#
    gcloud storage buckets create gs://$MY_BUCKET_NAME --location=$REGION
    
2. Copy a picture of a cat from a Google-provided Cloud Storage bucket to your Cloud Shell. You may use any picture of your choice.
#
    gcloud storage cp gs://cloud-training/ak8s/cat.jpg cat.jpg

3. Copy the image file to the bucket
#
    gcloud storage cp cat.jpg gs://$MY_BUCKET_NAME


4. Make bucket publicly accessible. This is an appropriate setting for hosting public website content in Cloud Storage.
#
    gsutil iam ch allUsers:objectViewer gs://$MY_BUCKET_NAME

You will see that the bucket is now publicly accessible.
Copy url of the image from the console and view it in a browser. You will see that the image can be viewed from your web browser.


## Hosting a static website on a google cloud compute engine.

1. Create vm in the default zone you created earlier. We will be making use of the default VPC for simplicity.
#
    gcloud compute instances create $MY_VM \
    --machine-type "e2-standard-2" \
    --image-project "debian-cloud" \
    --image-family "debian-11" \
    --subnet "default"
    
2. Verify the created VM
#
    gcloud compute instances list

3. Open firewall to allow http access and ssh


4. connect to vm via ssh by clicking on the ssh tab beside the VM on the google cloud console


5. Install nginx

#
    sudo apt-get update -y
    sudo apt-get install nginx -y

6. Open the editor tab in cloud shell and 
create an index.html file using the following text. Replace the string `REPLACE_WITH_CAT_URL` with the URL of the cat image from the previous task
#
    <html><head><title>Cat</title></head>
    <body>
    <h1>Cat</h1>
    <img src="REPLACE_WITH_CAT_URL">
    </body></html>

7. Copy the index.html file from cloudshell to the vm via scp

#
    gcloud compute scp index.html first-vm:index.nginx-debian.html --zone=us-central1-c


8. Copy the HTML file from your home directory to the document root of the nginx web server.

#
    sudo cp index.nginx-debian.html /var/www/html

9. Get the external IP of the VM from the console or terminal and open it in a web browser.





## Conclusion

We were able to host our static website on:
1. Publicly available storage bucket

2. Webserver on a VM