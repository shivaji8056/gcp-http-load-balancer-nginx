# gcp-http-load-balancer-nginx
"Step-by-step guide to setting up an HTTP Load Balancer with Nginx web servers on Google Cloud Platform. Includes fault-tolerant architecture using managed instance groups, health checks, and global load balancing."

# 🌐 GCP HTTP Load Balancer with Nginx Web Servers

## 📝 Objective
This repository provides a **step-by-step guide** to set up an **HTTP Load Balancer** on **Google Cloud Platform (GCP)** using **Nginx web servers**. The setup ensures fault tolerance and scalability by leveraging managed instance groups, health checks, and global load balancing.

---

## ⚙️ Prerequisites
1. Access to a **Google Cloud Platform (GCP)** project.
2. Basic knowledge of the **Google Cloud CLI (`gcloud`)**.
3. Permissions to create Compute Engine resources, firewall rules, and load balancers.

---

## 🚀 Steps to Set Up

### **Step 1: Create an Instance Template**
The instance template defines the configuration for Nginx web servers.

Run the following command:

```bash
gcloud compute instance-templates create nginx-template \
 --machine-type=e2-medium \
 --image-family=debian-11 \
 --image-project=debian-cloud \
 --metadata startup-script='#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- "s/nginx/Google Cloud Platform - $(hostname)/" /var/www/html/index.nginx-debian.html' \
 --tags=http-server,https-server
🔍 Verify the instance template:

gcloud compute instance-templates describe nginx-template
Step 2: Create a Managed Instance Group
Create a managed instance group with two instances based on the instance template.

gcloud compute instance-groups managed create nginx-group \
 --template=nginx-template \
 --size=2 \
 --zone=us-central1-f
🔍 Verify the instance group:

gcloud compute instance-groups managed list
gcloud compute instances list
Step 3: Create a Firewall Rule
Allow HTTP traffic on port 80 to the instances.

gcloud compute firewall-rules create grant-tcp-rule-207 \
 --allow=tcp:80 \
 --target-tags=http-server
🔍 Verify the firewall rule:

gcloud compute firewall-rules list
Step 4: Create a Health Check
Create an HTTP health check to monitor the health of the instances.

gcloud compute health-checks create http nginx-health-check \
 --request-path=/
🔍 Verify the health check:

gcloud compute health-checks describe nginx-health-check
Step 5: Create a Backend Service
Create a backend service and associate it with the health check.

gcloud compute backend-services create nginx-backend-service \
 --health-checks=nginx-health-check \
 --global
Add the instance group to the backend service:

gcloud compute instance-groups managed set-named-ports nginx-group \
 --named-ports http:80 \
 --zone us-central1-f

gcloud compute backend-services add-backend nginx-backend-service \
 --instance-group=nginx-group \
 --instance-group-zone=us-central1-f \
 --balancing-mode=UTILIZATION \
 --max-utilization=0.8 \
 --global
🔍 Verify the backend service:

gcloud compute backend-services describe nginx-backend-service --global
Step 6: Create a URL Map
Route incoming requests to the backend service.

gcloud compute url-maps create nginx-url-map \
 --default-service=nginx-backend-service
🔍 Verify the URL map:

gcloud compute url-maps describe nginx-url-map
Step 7: Create a Target HTTP Proxy
Create a target HTTP proxy to route requests to the URL map.

gcloud compute target-http-proxies create nginx-http-proxy \
 --url-map=nginx-url-map
🔍 Verify the target HTTP proxy:

gcloud compute target-http-proxies describe nginx-http-proxy
Step 8: Create a Forwarding Rule
Create a forwarding rule to direct traffic to the HTTP proxy.

gcloud compute forwarding-rules create nginx-forwarding-rule \
 --target-http-proxy=nginx-http-proxy \
 --global \
 --ports=80
🔍 Verify the forwarding rule:

gcloud compute forwarding-rules list
Step 9: Test Your Load Balancer
Get the external IP address of the forwarding rule:
gcloud compute forwarding-rules describe nginx-forwarding-rule --global
Look for the IPAddress field in the output.

Open the external IP address in your browser (e.g., http://<external-ip>). You should see the Nginx welcome page with the hostname of one of the instances.

Refresh the page multiple times to verify that the load balancer is distributing traffic between the two instances.

🎉 Conclusion
You have successfully set up an HTTP Load Balancer with Nginx web servers on Google Cloud Platform. This setup ensures fault tolerance and scalability for handling web traffic.
