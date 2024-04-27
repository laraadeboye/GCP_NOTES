#
    echo USER_2=[PROJECT ID]
    export PROJECT_ID=$(gcloud info --format='value(config.project)')

1. Create bucket
#
    gsutil mb -l us gs://$DEVSHELL_PROJECT_ID

2. Create file and enter text "I am amazing"
#
    cat > sample.txt <<EOF_END
    I am amazing
    EOF_END

3. Copy text to storage bucket
#
    gsutil cp sample.txt gs://$DEVSHELL_PROJECT_ID

4. Remove project viewer role

#
    gcloud projects remove-iam-policy-binding $DEVSHELL_PROJECT_ID --member=user:$USER_2 --role=roles/viewer

5. Add Storage object viewer role

#
    gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
    --role=roles/storage.objectViewer \
    --member=user:$USER_2


6. Create service account user named `read-bucket-objects`
#
    gcloud iam service-accounts create read-bucket-objects --display-name "read-bucket-objects" 

7. Add storage object viewer role
#
    gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member="serviceAccount:read-bucket-objects@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com" --role="roles/storage.objectViewer"


8. Add service account user role
#
    gcloud iam service-accounts add-iam-policy-binding  read-bucket-objects@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --member=domain:altostrat.com --role=roles/iam.serviceAccountUser

9. Add compute instance admin role
#
    gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member=domain:altostrat.com --role=roles/compute.instanceAdmin.v1


10. Create instance with service account role (read-bucket-object)
#
    gcloud compute instances create demoiam \
    --zone=$ZONE \
    --machine-type=e2-micro \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --service-account=read-bucket-objects@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com