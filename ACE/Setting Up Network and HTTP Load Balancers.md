# Setting Up Network and HTTP Load Balancers

    - In this lab , you will learn:
        - The differences between a network load balancer and a HTTP load balancer
        - How to configure Network and HTTP Load Balancers applications running on Google Compute Engine virtual machines

## L4 Network Load Balancer

## L7 HTTP(s) Load Balancer
ddd

## Configure Network and HTTP Load Balancers

* ### ***Set the default region and zone for all resources***

    In Cloud Shell, set the default zone:

    ```
    gcloud config set compute/zone us-central1-a

    ```
    set default region:

    ```
    gcloud config set compute/region us-central1
    ```
* ### ***Create multiple web server instances***

    To simulte, we will create a cluster of Nginx web servers using Instance Templates & Managed Instance Group
    
    Create a startup script to be used by every virtual machine instance in Cloud Shell.

    ```
    cat << EOF > startup.sh
    #! /bin/bash
    apt-get update
    apt-get install -y nginx
    service nginx start
    sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
    EOF
    ```

    Create an instance template, which uses the startup script:
    ```
    gcloud compute instance-templates create nginx-template --metadata-from-file stratup-script=startup.sh
    ```
    Create a target pool. A target pool allows a single access point to all the instances in a group and is necessary for load balancing in the future steps.
    
    ```
    gcloud compute target-pools create nginx-pool
    ```

    Create a managed instance group using ***the instance template*** & ***2 virtual machine instances*** with names that are prefixed with ***nginx-*** :
    ```
    gcloud compute instance-groups managed create nginx-group --base-instance-name nginx --size 2 --template nginx-template --target-pool nginx-pool 
    ```

    To see all of the instances created
    ```
    gcloud compute instances list
    ```
    
    Now configure a firewall so that you can connect to the machines on port 80 via the EXTERNAL_IP addresses:
    ```
    gcloud compute firewall-rules create www-firewall --allow tcp:80
    ```
* ## ***Create a Network Load Balancer***

    Create an L4 network load balancer targeting your instance group:
    ```
    gcloud compute forwarding-rules create nginx-lb \
    --region us-central1 \
    --ports=80 \
    --target-pool nginx-pool \
    ```

    List all Google Compute Engine forwarding rules in your project.
    ```
    gcloud compute forwarding-rules list
    ```

* ## ***Create a HTTP(s) Load Balancer***
    First, create a health check. Health checks verify that the instance is responding to HTTP or HTTPS traffic:
    ```
    gcloud compute http-health-checks create http-basic-check
    ```

    Define an HTTP service and map a port name to the relevant port for the instance group. Now the load balancing service can forward traffic to the named port:
    ```
    gcloud compute instance-groups managed \
       set-named-ports nginx-group \
       --named-ports http:80
    ```

    Create a backend service:
    ```
    gcloud compute backend-services create nginx-backend \
      --protocol HTTP --http-health-checks http-basic-check --global
    ```

    Add the instance group into the backend service:
    ```
    gcloud compute backend-services add-backend nginx-backend \
    --instance-group nginx-group \
    --instance-group-zone us-central1-a \
    --global
    ```

    Create a default URL map that directs all incoming requests to all your instances:
    ```
    gcloud compute url-maps create web-map \
    --default-service nginx-backend
    ```

    Create a target HTTP proxy to route requests to your URL map:
    ```
    gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map
    ```

    Create a global forwarding rule to handle and route incoming requests. A forwarding rule sends traffic to a specific target HTTP or HTTPS proxy depending on the IP address, IP protocol, and port specified. The global forwarding rule does not support multiple ports.
    ```
    gcloud compute forwarding-rules create http-content-rule \
        --global \
        --target-http-proxy http-lb-proxy \
        --ports 80
    ```

    After creating the global forwarding rule, it can take several minutes for your configuration to propagate.
    ```
    gcloud compute forwarding-rules list
    ```