# Essential Google Cloud Infrastructure: Foundation

# Implement Private Google Access and Cloud NAT

## Objectives

- In this lab, you learn how to perform the following tasks:
  - Configure a VM instance that doesn't have an external IP address
  - Connect to a VM instance using an Identity-Aware Proxy (IAP) tunnel
  - Enable Private Google Access on a subnet
  - Configure a Cloud NAT gateway
  - Verify access to public IP addresses of Google APIs and services and other connections to the internet

# Create the VM instance

## Create a VPC network and firewall rules

- Create VPC network privatenet.

  ```
  gcloud compute networks create privatenet --subnet-mode=custom
  ```

- To create the privatenet-us subnet, run the following command:

  ```
  gcloud compute networks subnets create privatenet-us --network=privatenet --region=us-central1 --range=10.130.0.0/20
  ```

- Create firewall rule.

  ```
  gcloud compute firewall-rules create privatenet-allow-ssh --network=privatenet --direction=INGRESS --action=ALLOW --rules=tcp:22 --source-ranges=35.235.240.0/20
  ```

  Create the VM instance with no public IP address

  ```
  gcloud compute instances create vm-internal --zone=us-central1-c --machine-type=n1-standard-1 --subnet=privatenet-us --no-address --no-service-account --no-scopes
  ```

- SSH to vm-internal to test the IAP tunnel:

  To connect to vm-internal, run the following command:

  ```
  gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap
  ```

  If prompted about continuing, type Y.
  When prompted for a passphrase, press ENTER.
  When prompted for the same passphrase, press ENTER.

  To test the external connectivity of vm-internal, run the following command:

  ```
  ping -c 2 www.google.com
  ```

  This should not work because vm-internal has no external IP address!

  Wait for the ping command to complete.

  To return to your Cloud Shell instance, run the following command:

  ```
  exit
  ```

- Create a Cloud Storage bucket

  ```
  gsutil mb gs://qwiklabs-gcp-04-6390b5af9637
  ```

  Copy an image file into your bucket

  ```
  gsutil cp gs://cloud-training/gcpnet/private/access.svg gs://qwiklabs-gcp-04-6390b5af9637
  ```

  In Cloud Shell, to try to copy the image from your bucket, run the following command:

  ```
  gsutil cp gs://qwiklabs-gcp-04-6390b5af9637/*.svg .
  ```

  This should work because Cloud Shell has an external IP address!

  To connect to vm-internal, run the following command:

  ```
  gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap
  ```

  If prompted, type Y to continue.

  To try to copy the image to vm-internal, run the following command:

  ```
  gsutil cp gs://qwiklabs-gcp-04-6390b5af9637/*.svg .
  ```

  This should not work: vm-internal can only send traffic within the VPC network because Private Google Access is disabled (by default).

- Enable Private Google Access

  ```
  gcloud compute networks subnets update privatenet-us --enable-private-ip-google-access --region=us-central1
  ```

  In Cloud Shell for vm-internal, to try to copy the image to vm-internal:

  ```
  gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap
  ```

  ```
  gsutil cp gs://qwiklabs-gcp-04-6390b5af9637/*.svg .
  ```

  This should work because vm-internal's subnet has Private Google Access enabled!

  To return to your Cloud Shell instance, run the following command:

  ```
  exit
  ```

## Configure a Cloud NAT gateway

- Configure a Cloud NAT gateway

  1. Creating Cloud Routers

  ```
  gcloud compute routers create nat-router --network privatenet --region us-central1
  ```

  2. Creating Cloud NAT

  ```
  gcloud compute routers nats create nat-config --router=nat-router --region=us-central1 --auto-allocate-nat-external-ips --nat-custom-subnet-ip-ranges=privatenet-us
  ```

  3. Verify the Cloud NAT gateway:

  In Cloud Shell for vm-internal, to try to re-synchronize the package index of vm-internal, run the following command:

  ```
  sudo apt-get update
  ```

  This should work because vm-internal is using the NAT gateway!

  To return to your Cloud Shell instance, run the following command:

  ```
  exit
  ```
