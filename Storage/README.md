
# Basic Gcloud Storage commands

1. create a bucket:
#
    gsutil mb -p [PROJECT_ID] gs://[BUCKET_NAME]


2. To get the default access list that's been assigned to setup.html, run the following command:
#
    gsutil acl get gs://$BUCKET_NAME_1/setup.html  > acl.txt 
    cat acl.txt


3. To set the access list to private and verify the results, run the following commands:
#
    gsutil acl set private gs://$BUCKET_NAME_1/setup.html
    gsutil acl get gs://$BUCKET_NAME_1/setup.html  > acl2.txt
    cat acl2.txt

4. To update the access list to make the file publicly readable, run the following commands:
#
    gsutil acl ch -u AllUsers:R gs://$BUCKET_NAME_1/setup.html
    gsutil acl get gs://$BUCKET_NAME_1/setup.html  > acl3.txt
    cat acl3.txt

5. Copy file named `setup.html` from bucket to the terminal:
#
    gcloud storage cp gs://$BUCKET_NAME_1/setup.html setup.html

6. Copy files from terminal to bucket:
#
    gsutil cp setup2.html gs://$BUCKET_NAME_1/
    gsutil cp setup3.html gs://$BUCKET_NAME_1/

7. Create service account storage object viewer
#
    gcloud iam service-accounts create cross-project-storage --display-name "Cross-Project Storage Account"

#
    gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member="serviceAccount:cross-project-storage@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com" --role="roles/storage.objectViewer"

8. Generate and download the JSON key file
#
    gcloud iam service-accounts keys create credentials.json --iam-account=cross-project-storage@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com

9. List files in a bucket:
#
    gcloud storage ls gs://$BUCKET_NAME_2/
## How to generate a Customer Supplied Encryption Key (CSEK)

10. Generate a base 64 encoded key:
#
    python3 -c 'import base64; import os; print(base64.encodebytes(os.urandom(32)))'

or store it in an Env variable:
#
    CSEK_KEY=$(python3 -c 'import base64; import os; print(base64.encodebytes(os.urandom(32)))')

11. Generate a new .boto file using the gsutil config -n command:
#
    gsutil config -n


Open .boto file and replace the encryption key line with the generated encryption key. (Leave out the b' and \n' at the end)


12. Add multiple files to the current directory (regex):
#
    gsutil cp gs://$BUCKET_NAME_1/setup* ./


## How to rotate CSEK keys

13. Rewrite file with new key:
#
    gsutil rewrite -k gs://$BUCKET_NAME_1/setup2.html

14. Review current lifecycle of bucket
#
    gsutil lifecycle get gs://$BUCKET_NAME_1

15. Set lifecycle policy
Create a file and paste the policy, then apply with the following command:
#
    gsutil lifecycle set life.json gs://$BUCKET_NAME_1


## Enable versioning

16. Review the current versioning status:
#
    gsutil versioning get gs://$BUCKET_NAME_1

17. Set versioning:
#
    gsutil versioning set on gs://$BUCKET_NAME_1

18. Create versioned file
#
    gcloud storage cp -v setup.html gs://$BUCKET_NAME_1

19. List all versions
#
    gcloud storage ls -a gs://$BUCKET_NAME_1/setup.html

20. sync a directory to a bucket
#
    gsutil rsync -r ./firstlevel gs://$BUCKET_NAME_1/firstlevel




# Basic Gcloud Storage commands

1. create a bucket:
#
    gsutil mb -p [PROJECT_ID] gs://[BUCKET_NAME]


2. To get the default access list that's been assigned to setup.html, run the following command:
#
    gsutil acl get gs://$BUCKET_NAME_1/setup.html  > acl.txt 
    cat acl.txt


3. To set the access list to private and verify the results, run the following commands:
#
    gsutil acl set private gs://$BUCKET_NAME_1/setup.html
    gsutil acl get gs://$BUCKET_NAME_1/setup.html  > acl2.txt
    cat acl2.txt

4. To update the access list to make the file publicly readable, run the following commands:
#
    gsutil acl ch -u AllUsers:R gs://$BUCKET_NAME_1/setup.html
    gsutil acl get gs://$BUCKET_NAME_1/setup.html  > acl3.txt
    cat acl3.txt

5. Copy file named `setup.html` from bucket to the terminal:
#
    gcloud storage cp gs://$BUCKET_NAME_1/setup.html setup.html

6. Copy files from terminal to bucket:
#
    gsutil cp setup2.html gs://$BUCKET_NAME_1/
    gsutil cp setup3.html gs://$BUCKET_NAME_1/

7. Create service account storage object viewer
#
    gcloud iam service-accounts create cross-project-storage --display-name "Cross-Project Storage Account"

#
    gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member="serviceAccount:cross-project-storage@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com" --role="roles/storage.objectViewer"

8. Generate and download the JSON key file
#
    gcloud iam service-accounts keys create credentials.json --iam-account=cross-project-storage@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com

9. List files in a bucket:
#
    gcloud storage ls gs://$BUCKET_NAME_2/
## How to generate a Customer Supplied Encryption Key (CSEK)

10. Generate a base 64 encoded key:
#
    python3 -c 'import base64; import os; print(base64.encodebytes(os.urandom(32)))'

or store it in an Env variable:
#
    CSEK_KEY=$(python3 -c 'import base64; import os; print(base64.encodebytes(os.urandom(32)))')

11. Generate a new .boto file using the gsutil config -n command:
#
    gsutil config -n


Open .boto file and replace the encryption key line with the generated encryption key. (Leave out the b' and \n' at the end)


12. Add multiple files to the current directory (regex):
#
    gsutil cp gs://$BUCKET_NAME_1/setup* ./


## How to rotate CSEK keys

13. Rewrite file with new key:
#
    gsutil rewrite -k gs://$BUCKET_NAME_1/setup2.html

14. Review current lifecycle of bucket
#
    gsutil lifecycle get gs://$BUCKET_NAME_1

15. Set lifecycle policy
Create a file and paste the policy, then apply with the following command:
#
    gsutil lifecycle set life.json gs://$BUCKET_NAME_1


## Enable versioning

16. Review the current versioning status:
#
    gsutil versioning get gs://$BUCKET_NAME_1

17. Set versioning:
#
    gsutil versioning set on gs://$BUCKET_NAME_1

18. Create versioned file
#
    gcloud storage cp -v setup.html gs://$BUCKET_NAME_1

19. List all versions
#
    gcloud storage ls -a gs://$BUCKET_NAME_1/setup.html

20. sync a directory to a bucket
#
    gsutil rsync -r ./firstlevel gs://$BUCKET_NAME_1/firstlevel



