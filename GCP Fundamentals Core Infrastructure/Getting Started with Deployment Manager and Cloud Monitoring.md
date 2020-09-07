# Google Cloud Fundamentals: Getting Started with Deployment Manager and Cloud Monitoring

## Objectives

- In this lab, We learn how to perform the following tasks:
  - Create a Deployment Manager deployment.
  - Update a Deployment Manager deployment.
  - View the load on a VM instance using Cloud Monitoring.

## Create a Deployment Manager deployment

At the Cloud Shell
place the zone that Qwiklabs assigned you to into an environment variable called MY_ZONE.

```
export MY_ZONE=us-central1-a
```

Download an editable Deployment Manager template:

```
gsutil cp gs://cloud-training/gcpfcoreinfra/mydeploy.yaml mydeploy.yaml
```

we use the sed command to replace the PROJECT_ID placeholder string with your Google Cloud Platform project ID using this command:

```
sed -i -e "s/PROJECT_ID/$DEVSHELL_PROJECT_ID/" mydeploy.yaml
```

we use the sed command to replace the ZONE placeholder string with your Google Cloud Platform zone using this command:

```
sed -i -e "s/ZONE/$MY_ZONE/" mydeploy.yaml
```

View the mydeploy.yaml file, with your modifications, with this command:

```
cat mydeploy.yaml
```

The file will look something like this:

```yaml
resources:
  - name: my-vm
    type: compute.v1.instance
    properties:
      zone: us-central1-a
      machineType: zones/us-central1-a/machineTypes/n1-standard-1
      metadata:
        items:
          - key: startup-script
            value: "apt-get update"
      disks:
        - deviceName: boot
          type: PERSISTENT
          boot: true
          autoDelete: true
          initializeParams:
            sourceImage: https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/debian-9-stretch-v20180806
      networkInterfaces:
        - network: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-dcdf854d278b50cd/global/networks/default
          accessConfigs:
            - name: External NAT
              type: ONE_TO_ONE_NAT
```

Build a deployment from the template:

```
gcloud deployment-manager deployments create my-first-depl --config mydeploy.yaml
```

Confirm that the deployment was successful. we list all compute instance, with this command:

```
gcloud compute instances list
```

You will see that a VM instance called my-vm has been created

To Confirm that the startup script we specified in Deployment Manager template has been installed

```
gcloud compute instances describe my-vm --zone=us-central1-a
```

Scroll down to the metadata section and confirm that the startup script

```yaml
metadata:
  fingerprint: 17j-1k4N_F8=
  items:
    - key: startup-script
      value: apt-get update;
  kind: compute#metadata
```

## Update a Deployment Manager deployment

Return to your Cloud Shell prompt. Launch the nano text editor to edit the mydeploy.yaml file:

```
nano mydeploy.yaml
```

Find the line that sets the value of the startup script, value: "apt-get update", and edit it so that it looks like this:

```
    value: "apt-get update; apt-get install nginx-light -y"
```

**NB**:
Do not disturb the spaces at the beginning of the line. The YAML templating language relies on indented lines as part of its syntax. As you edit the file, be sure that the v in the word value in this new line is immediately below the k in the word key on the line above it.

Press Ctrl+O and then press Enter to save your edited file.
Press Ctrl+X to exit the nano text editor.

Return to your Cloud Shell prompt. Enter this command to cause Deployment Manager to update your deployment to install the new startup script:

```
gcloud deployment-manager deployments update my-first-depl --config mydeploy.yaml
```

To Confirm that the startup script we specified in Deployment Manager template has been updated

```
gcloud compute instances describe my-vm --zone=us-central1-a
```

Scroll down to the metadata section and confirm that the startup script

```yaml
metadata:
  fingerprint: 17j-1k4N_F8=
  items:
    - key: startup-script
      value: apt-get update; apt-get install nginx-light -y
  kind: compute#metadata
```

## View the Load on a VM using Cloud Monitoring

SSH into virtual machine instance my-vm-2

```
gcloud compute ssh my-vm --zone=us-central1-b
```

In the ssh session on my-vm, execute this command to create a CPU load:

```
dd if=/dev/urandom | gzip -9 >> /dev/null &
```

This Linux pipeline forces the CPU to work on compressing a continuous stream of random data.

- **_Installing the Cloud Monitoring agent on a single VM:_**

  - Make sure you have sudo access:

  ```
  sudo -sH
  ```

  - Add the agent's package repository:

  ```
  curl -sSO https://dl.google.com/cloudagents/add-monitoring-agent-repo.sh
  sudo bash add-monitoring-agent-repo.sh
  sudo apt-get update
  ```

  - Install the agent:

  ```
    sudo apt-get install stackdriver-agent
  ```

  - Start the agent service:

  ```
    sudo service stackdriver-agent start
  ```

- **_Installing the Cloud Logging agent on a single VM:_**

  - Make sure you have sudo access:

  ```
  sudo -sH
  ```

  - Add the agent's package repository:

  ```
  curl -sSO https://dl.google.com/cloudagents/add-logging-agent-repo.sh
  sudo bash add-logging-agent-repo.sh
  sudo apt-get update
  ```

  - Install the agent:

  ```
      sudo apt-get install google-fluentd
  ```

  - Install the Configuration files.

    - For unstructured logging, run:

      ```
        sudo apt-get install -y google-fluentd-catch-all-config
      ```

    - For structured logging, run:

      ```
      sudo apt-get install -y google-fluentd-catch-all-config-structured
      ```

- Start the agent service:

```
  sudo service google-fluentd start
```

In the Metric pane of Metrics Explorer, select the resource type GCE VM instance and the metric CPU usage.

In the resulting graph, notice that CPU usage increased sharply a few minutes ago.

Terminate your workload generator. Return to your ssh session on my-vm and enter this command:

```
kill %1
```
