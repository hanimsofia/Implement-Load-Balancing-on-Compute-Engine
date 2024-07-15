# Implement Load Balancing on Compute Engine: Challenge Lab

## Challenge scenario
You have started a new role as a Junior Cloud Engineer for Jooli, Inc. You are expected to help manage the infrastructure at Jooli. Common tasks include provisioning resources for projects.

You are expected to have the skills and knowledge for these tasks, so step-by-step guides are not provided.

Some Jooli, Inc. standards you should follow:

 * Create all resources in the default region or zone, unless otherwise directed. The default region is **REGION**, and the default zone is **ZONE**.
 * Naming normally uses the format team-resource; for example, an instance could be named nucleus-webserver1.
 * Allocate cost-effective resource sizes. Projects are monitored, and excessive resource use will result in the containing project's termination (and possibly yours), so plan carefully. This is the guidance the monitoring team is willing to share: unless directed, use e2-micro for small Linux VMs, and use e2-medium for Windows or other applications, such as Kubernetes nodes.

### Task 1. Create a project jumphost instance
You will use this instance to perform maintenance for the project.

Requirements:

 * Name the instance **Instance name**.
 * Create the instance in the **ZONE** zone.
 * Use an e2-micro machine type.
 * Use the default image type (Debian Linux).

<details>
<summary>Task 1 Answer</summary>
<br>
        
```
gcloud config set compute/region REGION

export REGION=REGION

export ZONE=Zone

gcloud compute instances create **<INSTANCE NAME>** –machine-type e2-micro --zone=**<ZONE>**
```

</details>

### Task 2. Create a Kubernetes service cluster
Note: There is a limit to the resources you are allowed to create in your project. If you don't get the result you expected, delete the cluster before you create another cluster. If you don't, the lab might end and you might be blocked. To get your account unblocked, you will have to reach out to Google Cloud Skills Boost Support.
The team is building an application that will use a service running on Kubernetes. You need to:

 * Create a zonal cluster using ZONE.
 * Use the Docker container hello-app (gcr.io/google-samples/hello-app:2.0) as a placeholder; the team will replace the container with their own work later.
 * Expose the app on port App port number.

<details>
<summary>Task 2 Answer</summary>
<br>
        
```
gcloud container clusters create --machine-type=e2-medium --zone=ZONE lab-cluster
kubectl create deployment hello-server --image= gcr.io/google-samples/hello-app:2.0
kubectl expose deployment hello-server --type=LoadBalancer --port 8081
kubectl get service 
```

</details>

### Task 3. Set up an HTTP load balancer
You will serve the site via nginx web servers, but you want to ensure that the environment is fault-tolerant. Create an HTTP load balancer with a managed instance group of 2 nginx web servers. Use the following code to configure the web servers; the team will replace this with their own configuration later.
        
        cat << EOF > startup.sh
        #! /bin/bash
        apt-get update
        apt-get install -y nginx
        service nginx start
        sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
        EOF

Note: There is a limit to the resources you are allowed to create in your project, so do not create more than 2 instances in your managed instance group. If you do, the lab might end and you might be banned.

You need to:

 * Create an instance template. Don't use the default machine type. Make sure you specify e2-medium as the machine type.
 * Create a target pool.
 * Create a managed instance group.
 * Create a firewall rule named as **Firewall rule** to allow traffic (80/tcp).
 * Create a health check.
 * Create a backend service, and attach the managed instance group with named port (http:80).
 * Create a URL map, and target the HTTP proxy to route requests to your URL map.
 * Create a forwarding rule.

Note: You may need to wait for 5 to 7 minutes to get the score for this task.

<details>
<summary>Task 3 Answer</summary>
<br>

Create the web server frontend:

```
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
 
gcloud compute instance-templates create web-server-template \
        --metadata-from-file startup-script=startup.sh \
 
        --machine-type g1-small \
        --region <dynamic-region>
 
gcloud compute instance-groups managed create web-server-group \
        --base-instance-name web-server \
        --size 2 \
        --template web-server-template \
        --region <dynamic-region>
 
gcloud compute firewall-rules create <dynamic-firewall-rule> \
        --allow tcp:80 \
        --network default
 
gcloud compute http-health-checks create http-basic-check
 
gcloud compute instance-groups managed \
        set-named-ports web-server-group \
        --named-ports http:80 \
        --region <dynamic-region>
 
gcloud compute backend-services create web-server-backend \
        --protocol HTTP \
        --http-health-checks http-basic-check \
        --global
 
gcloud compute backend-services add-backend web-server-backend \
        --instance-group web-server-group \
        --instance-group-region <dynamic-region> \
        --global
 
gcloud compute url-maps create web-server-map \
        --default-service web-server-backend
 
gcloud compute target-http-proxies create http-lb-proxy \
        --url-map web-server-map
 
gcloud compute forwarding-rules create http-content-rule \
      --global \
      --target-http-proxy http-lb-proxy \
      --ports 80
 
gcloud compute forwarding-rules list
```
</details>


## Sample Answer
Checkout the collapsed section for each task!! :star:
