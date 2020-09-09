# Essential Google Cloud Infrastructure: Foundation

# Lab: VPC Networking

## Objectives

- In this lab, We are going to learn how to perform the following tasks:
  - Explore the default VPC network
  - Create an auto mode network with firewall rules
  - Convert an auto mode network to a custom mode network
  - Create custom mode VPC networks with firewall rules
  - Create VM instances using Compute Engine
  - Explore the connectivity for VM instances across VPC networks

# Explore the default network

- View networks
  ```
  gcloud compute networks list
  ```
- View the subnets

  list displays all Google Compute Engine subnetworks in a project.

  ```
  gcloud compute networks subnets list
  ```

  add `--network=NETWORK` to show only subnetworks of a specific network.

  ```
  gcloud compute networks subnets list --network=NETWORK
  ```

  ```
    gcloud compute networks subnets list --network=default
  ```

- View the routes

  ```
    gcloud compute routes list
  ```

- View the firewall rules

  ```
    gcloud compute firewall-rules list
  ```

- Delete the Firewall rules
  delete all default network firewall rules

  ```
    gcloud compute firewall-rules delete default-allow-icmp default-allow-internal default-allow-rdp default-allow-ssh
  ```

- Delete the default network
  ```
  gcloud compute networks delete default
  ```

## Create an auto mode VPC network with firewall rules

- Create an auto mode VPC network

  ```
  gcloud compute networks create mynetwork --subnet-mode=auto --bgp-routing-mode=regional
  ```

  `Instances on this network will not be reachable until firewall rules are created. As an example, you can allow all internal traffic between instances as well as SSH, RDP, and ICMP by running:`

  ```
  $ gcloud compute firewall-rules create <FIREWALL_NAME> --network mynetwork --allow tcp,udp,icmp --source-ranges <IP_RANGE>
  ```

  ```
  $ gcloud compute firewall-rules create <FIREWALL_NAME> --network mynetwork --allow tcp:22,tcp:3389,icmp
  ```

- Create firewall rules

  - Allows ICMP connections from any source to any instance on the network

    ```
    gcloud compute firewall-rules create mynetwork-allow-icmp --network=mynetwork --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=icmp

    ```

  - Allows connections from any source in the network IP range to any instance on the network using all protocols.

    ```
    gcloud compute firewall-rules create mynetwork-allow-internal   --network=mynetwork --direction=INGRESS --priority=65534 --source-ranges=10.128.0.0/9 --action=ALLOW --rules=all
    ```

  - Allows RDP connections from any source to any instance on the network using port 3389.

    ```
    gcloud compute firewall-rules create mynetwork-allow-rdp --network=mynetwork  --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:3389
    ```

  - Allows TCP connections from any source to any instance on the network using port 22.

    ```
    gcloud compute firewall-rules create mynetwork-allow-ssh   --network=mynetwork --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:22
    ```

_**Record the IP address range for the subnets in us-central1 and europe-west1. These will be referred to in the next steps.**_

```
gcloud compute networks subnets list --network=mynetwork --filter="region:( us-central1, europe-west1)"
```

```
NAME REGION NETWORK RANGE
mynetwork us-central1 mynetwork 10.128.0.0/20
mynetwork europe-west1 mynetwork 10.132.0.0/20
```

## Create a VM instance in us-central1

```
gcloud compute instances create mynet-us-vm --zone=us-central1-c --machine-type=n1-standard-1 --subnet=mynetwork
```

```
gcloud compute instances list --filter=name=mynet-us-vm
```

```
NAME ZONE MACHINE_TYPE PREEMPTIBLE INTERNAL_IP EXTERNAL_IP STATUS
mynet-us-vm us-central1-c n1-standard-1 10.128.0.2 35.225.251.200 RUNNING
```

```
gcloud compute instances create mynet-eu-vm --zone=europe-west1-c --machine-type=n1-standard-1 --subnet=mynetwork
```

```
gcloud compute instances list --filter=name=mynet-eu-vm
```

```
NAME ZONE MACHINE_TYPE PREEMPTIBLE INTERNAL_IP EXTERNAL_IP STATUS
mynet-eu-vm europe-west1-c n1-standard-1 10.132.0.2 34.77.113.224 RUNNING
```

```
gcloud compute ssh mynet-us-vm --zone=us-central1-c
```

## Convert the network to a custom mode network

Network [mynetwork] will be switched to custom mode. This operation
cannot be undone.

```
gcloud compute networks update mynetwork --switch-to-custom-subnet-mode
```

## Create the managementnet network

To create the managementnet network, run the following command:

```
gcloud compute networks create managementnet --subnet-mode=custom --bgp-routing-mode=regional
```

To create the managementsubnet-us subnet, run the following command:

```
gcloud compute networks subnets create managementsubnet-us --range=10.130.0.0/20 --network=managementnet --region=us-central1
```

## Create the privatenet network

To create the privatenet network, run the following command:

```
gcloud compute networks create privatenet --subnet-mode=custom
```

To create the privatesubnet-us subnet, run the following command:

```
gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/24
```

To create the privatesubnet-eu subnet, run the following command:

```
gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west1 --range=172.20.0.0/20
```

To list the available VPC networks, run the following command:

```
gcloud compute networks list
```

## Create the firewall rules for managementnet

Create firewall rules to allow SSH, ICMP, and RDP ingress traffic to VM instances on the managementnet network.

```
gcloud compute firewall-rules create managementnet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=tcp:22,tcp:3389,icmp --source-ranges=0.0.0.0/0
```

## Create the firewall rules for privatenet

Create firewall rules to allow SSH, ICMP, and RDP ingress traffic to VM instances on the privatenet network.

```
gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0
```

To list all the firewall rules (sorted by VPC network), run the following command:

```
gcloud compute firewall-rules list --sort-by=NETWORK
```

result:

```
NAME NETWORK DIRECTION PRIORITY ALLOW DENY DISABLED
managementnet-allow-icmp-ssh-rdp managementnet INGRESS 1000 tcp:22,tcp:3389,icmp False
mynetwork-allow-icmp mynetwork INGRESS 65534 icmp False
mynetwork-allow-internal mynetwork INGRESS 65534 all False
mynetwork-allow-rdp mynetwork INGRESS 65534 tcp:3389 False
mynetwork-allow-ssh mynetwork INGRESS 65534 tcp:22 False
privatenet-allow-icmp-ssh-rdp privatenet INGRESS 1000 icmp,tcp:22,tcp:3389 False
```

## Create the managementnet-us-vm instance

Create the managementnet-us-vm instance using the gcloud command line.

```
gcloud compute instances create managementnet-us-vm --zone=us-central1-c --machine-type=f1-micro --subnet=managementsubnet-us
```

Create the privatenet-us-vm instance using the gcloud command line.

```
gcloud compute instances create privatenet-us-vm --zone=us-central1-c --machine-type=f1-micro --subnet=privatesubnet-us
```

To list all the VM instances (sorted by zone), run the following command:

```
gcloud compute instances list --sort-by=ZONE --format='table(name,ZONE,INTERNAL_IP,EXTERNAL_IP,status,networkInterfaces[].network.notnull().list().segment(9):label=NETWORK)'
```

| NAME                | ZONE           | INTERNAL_IP | EXTERNAL_IP    | STATUS  | NETWORK       |
| ------------------- | -------------- | ----------- | -------------- | ------- | ------------- |
| mynet-eu-vm         | europe-west1-c | 10.132.0.2  | 34.77.113.224  | RUNNING | mynetwork     |
| managementnet-us-vm | us-central1-c  | 10.130.0.2  | 35.232.215.137 | RUNNING | managementnet |
| mynet-us-vm         | us-central1-c  | 10.128.0.2  | 35.225.251.200 | RUNNING | mynetwork     |
| privatenet-us-vm    | us-central1-c  | 172.16.0.2  | 34.122.137.163 | RUNNING | privatenet    |

## Explore the connectivity across networks

- SHH into mynet-us-vm, run the following command:

  ```
  gcloud compute ssh mynet-us-vm --zone=us-central1-c
  ```

- To test connectivity to mynet-eu-vm's external IP, run the following command, replacing mynet-eu-vm's external IP:

  ```
  ping -c 3 <Enter mynet-eu-vm's external IP here>
  ```

  This should work!

- To test connectivity to managementnet-us-vm's external IP, run the following command, replacing managementnet-us-vm's external IP:

  ```
  ping -c 3 <Enter managementnet-us-vm's external IP here>
  ```

  This should work!

- To test connectivity to privatenet-us-vm's external IP, run the following command, replacing privatenet-us-vm's external IP:

  ```
  ping -c 3 <Enter privatenet-us-vm's external IP here>
  ```

  This should work!

  You can ping the external IP address of all VM instances, even though they are in either a different zone or VPC network. This confirms that public access to those instances is only controlled by the ICMP firewall rules that you established earlier.

- To test connectivity to mynet-eu-vm's internal IP, run the following command, replacing mynet-eu-vm's internal IP:

  ```
  ping -c 3 <Enter mynet-eu-vm's internal IP here>
  ```

  You can ping the internal IP address of mynet-eu-vm because it is on the same VPC network as the source of the ping (mynet-us-vm), even though both VM instances are in separate zones, regions, and continents!

- To test connectivity to managementnet-us-vm's internal IP, run the following command, replacing managementnet-us-vm's internal IP:

  ```
  ping -c 3 <Enter managementnet-us-vm's internal IP here>
  ```

  This should not work, as indicated by a 100% packet loss!

- To test connectivity to privatenet-us-vm's internal IP, run the following command, replacing privatenet-us-vm's internal IP:

  ```
  ping -c 3 <Enter privatenet-us-vm's internal IP here>
  ```

  This should not work either, as indicated by a 100% packet loss! You cannot ping the internal IP address of managementnet-us-vm and privatenet-us-vm because they are in separate VPC networks from the source of the ping (mynet-us-vm), even though they are all in the same zone, us-central1-c.
