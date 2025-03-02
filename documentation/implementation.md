### **📌 `implementation.md`** (Step-by-Step Documentation of What I Did)  

# HAProxy Load Balancer Implementation

## **Overview**
For this project, I set up **HAProxy Load Balancing** using **three AWS EC2 instances** on the **AWS cloud platform** as my VMs.  

- **VM 1**: Web Server 1 (**Nginx**)  
- **VM 2**: Web Server 2 (**Nginx**)  
- **VM 3**: Load Balancer (**HAProxy**)  

The goal was to distribute traffic efficiently between the two web servers using HAProxy and test different **load balancing algorithms**:
1. **Round Robin**
2. **Least Connections**
3. **IP Hash**

---

## **1️⃣ Deploying AWS EC2 Instances**
I launched three **Amazon Linux 2 / Ubuntu** EC2 instances:  
- **Two for web servers**
- **One for the HAProxy load balancer**

### **Steps I Followed**
1. **Created three EC2 instances** from the AWS console.
2. Used **t2.micro** instances (since they are free-tier eligible).
3. Configured **security group rules**:
   - **Allowed SSH (22)**
   - **Allowed HTTP (80)**
   - **Allowed HAProxy traffic (80)**
   - **Allowed traffic for the first webserver (8081)**
   - **Allowed traffic for the second webserver (8082)**

4. Assigned **Elastic IPs** to keep the IP addresses static.

---

## **2️⃣ Configuring the Web Servers**
I configured **two web servers** with **Nginx**, each serving different content.

### **Setting Up Web Server 1**
Logged into the first EC2 instance and ran:
```bash
sudo apt update && sudo apt install nginx -y
echo "<h1>Welcome to Web Server 1</h1>" | sudo tee /var/www/html/index.html
sudo systemctl restart nginx
```

### **Setting Up Web Server 2**
Logged into the second EC2 instance and ran:
```bash
sudo apt update && sudo apt install nginx -y
echo "<h1>Welcome to Web Server 2</h1>" | sudo tee /var/www/html/index.html
sudo systemctl restart nginx
```

### **Verification**
I tested both instances by opening their **public IPs** in a browser:
```
http://<Web-Server-1-IP>
http://<Web-Server-2-IP>
```
Each server displayed its respective content.

---

## **3️⃣ Installing and Configuring HAProxy**
For the load balancer, I used a third EC2 instance.

### **Installing HAProxy**
After logging into the **HAProxy VM**, I installed HAProxy:
```bash
sudo apt update && sudo apt install haproxy -y
```

### **Editing the HAProxy Configuration**
I modified `/etc/haproxy/haproxy.cfg`:
```bash
sudo nano /etc/haproxy/haproxy.cfg
```

Then I added:
```haproxy
frontend http_front
    bind *:80
    mode http
    default_backend backend_servers

backend backend_servers
    balance roundrobin  # Can be changed to leastconn or ip_hash
    server server1 <Web-Server-1-IP>:8081 check
    server server2 <Web-Server-2-IP>:8082 check
```

### **Restarting HAProxy**
```bash
sudo systemctl restart haproxy
```

---

## **4️⃣ Configuring Health Checks**
I tested HAProxy’s health check by checking HAProxy logs:
```bash
tail -f /var/log/haproxy.log
```
It was marked as **UP** meaning everything was working perfectly.

---

## **5️⃣ Testing Load Balancing**
I accessed the **HAProxy Load Balancer** via:
```
http://<Load-Balancer-IP>
```
Refreshing the page showed responses from **both servers**, confirming traffic was being distributed.

---

## **Next Steps: Load Testing**
To verify load balancing efficiency, I used `h2load` for performance testing (documented in `testing.md`).

✅ **HAProxy Load Balancer was successfully implemented and tested!**
