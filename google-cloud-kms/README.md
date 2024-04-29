
# Google Cloud KMS

Overview

In order to encrypt the data, you need to create a KeyRing and a CryptoKey. KeyRings are useful for grouping keys. Keys can be grouped by environment (like test, staging, and prod) or by some other conceptual grouping. 
In KMS, there are two major permissions to focus on. One permissions allows a user or service account to manage KMS resources, the other allows a user or service account to use keys to encrypt and decrypt data.

The permission to manage keys is `cloudkms.admin`, and allows anyone with the permission to create KeyRings and create, modify, disable, and destroy CryptoKeys. The permission to encrypt and decrypt is `cloudkms.cryptoKeyEncrypterDecrypter`, and is used to call the encrypt and decrypt API endpoints.

## Common commands


1. Enable cloud KMS
#
    gcloud services enable cloudkms.googleapis.com

2. Assuming your KeyRing is called `test` and your CryptoKey is called `qwiklab`. To create a KeyRing and CryptoKey.
#
    KEYRING_NAME=test CRYPTOKEY_NAME=qwiklab

#
    gcloud kms keyrings create $KEYRING_NAME --location global
#
    gcloud kms keys create $CRYPTOKEY_NAME --location global \
    --keyring $KEYRING_NAME \
    --purpose encryption

3. Get current authorized user save as USER_EMAIL
#
    USER_EMAIL=$(gcloud auth list --limit=1 2>/dev/null | grep '@' | awk '{print $2}')

4. Assign IAM permissions to manage KeyRing
#
    gcloud kms keyrings add-iam-policy-binding $KEYRING_NAME \
    --location global \
    --member user:$USER_EMAIL \
    --role roles/cloudkms.admin
5. Assign permissions to decrypt
#
    gcloud kms keyrings add-iam-policy-binding $KEYRING_NAME \
    --location global \
    --member user:$USER_EMAIL \
    --role roles/cloudkms.cryptoKeyEncrypterDecrypter

6. Copy and encrypt files to a cloud storage bucket.Encrypt multiple files using KMS API and upload to Cloud Storage.

- Assume **allen-p** is a set of emails that you wan to encrypt whis is stored in a cloud storage bucket. First, copy all emails for allen-p into your current working directory:
#
    gsutil -m cp -r gs://enron_emails/allen-p . 

- then, run an encryption and back-up action with the following command:


```
MYDIR=allen-p
FILES=$(find $MYDIR -type f -not -name "*.encrypted")
for file in $FILES; do
  PLAINTEXT=$(cat $file | base64 -w0)
  curl -v "https://cloudkms.googleapis.com/v1/projects/$DEVSHELL_PROJECT_ID/locations/global/keyRings/$KEYRING_NAME/cryptoKeys/$CRYPTOKEY_NAME:encrypt" \
    -d "{\"plaintext\":\"$PLAINTEXT\"}" \
    -H "Authorization:Bearer $(gcloud auth application-default print-access-token)" \
    -H "Content-Type:application/json" \
  | jq .ciphertext -r > $file.encrypted
done
gsutil -m cp allen-p/inbox/*.encrypted gs://${BUCKET_NAME}/allen-p/inbox
```