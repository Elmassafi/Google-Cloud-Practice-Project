# Google Cloud Fundamentals:  Getting Started with Compute Engine


## Objectives 
In this lab, you will learn how to perform the following tasks:
- Create a Compute Engine virtual machine using the Google Cloud Platform (GCP) Console.
- Create a Compute Engine virtual machine using the gcloud command-line interface.
- Connect between the two instances.


## Create a Compute Engine virtual machine using the Google Cloud Platform (GCP) Console.

*from Console instructions to 100% command line instructions*

We will create a virtual machine in zone us-central1-a and we use default value for machine type and subnet and we will tag the virtual machine with **_http-server_** , tag will be used for firewall rules

```
gcloud compute instances create my-vm-1 --zone=us-central1-a --subnet=default --tags=http-server --image=debian-9-stretch-v20200805 --image-project=debian-cloud
```
***NB**: debian image can be find using the following command:*  
```
gcloud compute images list | grep debian-9
```

To allow HTTP traffic we will create a firewall that allows ingress TCP traffic on port 80 

```
gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
```

## Create a Compute Engine virtual machine using the gcloud command-line interface.

TO set your default zone to the one you just chose, enter this partial command gcloud config set compute/zone followed by the zone you chose.

```
gcloud config set compute/zone us-central1-b
```

Create a VM instance called my-vm-2 in that zone
```
gcloud compute instances create "my-vm-2" --machine-type "n1-standard-1" --image-project "debian-cloud" --image "debian-9-stretch-v20190213" --subnet "default"
```

## Connect between VM instances

Now we have two VM instances, each in a different zone.

1. SSH into  virtual machine instance my-vm-2 

```
gcloud compute ssh my-vm-2 --zone=us-central1-b
```

2. Use the ping command to confirm that my-vm-2 can reach my-vm-1 over the network

```
ping my-vm-1
```

3.  Use the ssh command to open a command prompt on my-vm-1:

```
ssh my-vm-1
```

4. At my-vm-1, we install the Nginx web server:

```
sudo apt-get install nginx-light -y
```

5.  use the nano text editor to add a custom message to the home page of the web server:

```
sudo nano /var/www/html/index.nginx-debian.html
```

6.  Use the arrow keys to move the cursor to the line just below the h1 header. Add text like this, and replace YOUR_NAME with your name:

```
Hi from EL MASSAFI
```

7.  Press **Ctrl+O** and then press Enter to save your edited file, and then press **Ctrl+X** to exit the nano text editor. 

8.  Confirm that the web server is serving your new page.

```
curl http://localhost/
```

***NB**: The response will be the HTML source of the web server's home page, including line Hi from EL MASSAFI.*

9.  we exit from my-vm-1 and let's confirm that **my-vm-2** can reach the web server on **my-vm-1**

```
curl http://my-vm-1/
```

***NB**: The response will be the HTML source of the web server's home page, including line Hi from EL MASSAFI.*

10. Copy the External IP address for my-vm-1 and paste it into the address bar of a new browser tab. We will see the web server's home page, including custom text.

- the External IP address for my-vm-1

```
gcloud compute instances list --filter=name=my-vm-1
OR
gcloud compute instances list --filter="name:(my-vm-1)"
```

***NB**: The web server's home page*

![The web server's home page](./images/web-server-home-page.png)

## End 
We created virtual machine (VM) instances in two different zones and connected to them using ping, ssh, and HTTP.