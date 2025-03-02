# Load-Balancer-Implementation

## **Project Overview**  
This project demonstrates how to set up **HAProxy** as a **load balancer** for distributing traffic between multiple web servers.  
We used **three AWS EC2 instances**:  
- **Two web servers** configured with **Nginx** serving different content.  
- **One HAProxy server** acting as a **load balancer**.  

The project tests and compares **three load balancing algorithms**:  
1. **Round Robin** – Equal distribution of traffic.  
2. **IP Hash** – Routes requests based on client IP.  
3. **Least Connections** – Sends traffic to the least loaded server.  

---

## **Infrastructure Setup 🏗️**  
### **Cloud Platform:** AWS EC2  
- **Instance 1 & 2**: Nginx web servers  
- **Instance 3**: HAProxy load balancer  

### **Configuration Summary**  
- **Web Servers (Nginx)** serve **different** content.  
- **HAProxy** is configured to distribute traffic using **three different algorithms**.  
- **Health checks** ensure only active servers receive requests.  

---

## **📂 Project Structure**  
```
haproxy-loadbalancing-lab/
├── configs/
│   └── haproxy.cfg
│
├── documentation/
│   ├── implementation.md
│   ├── testing.md
│   └── screenshots/
│
├── README.md
└── .gitignore
```

---

## **Step-by-Step Implementation**  
### **1️⃣ Deploy Web Servers (AWS EC2)**
I launched **two EC2 instances** and installed **Nginx**:  
```bash
sudo apt update
sudo apt install nginx -y
```
Each server was configured to serve **different content**.  

**Web Server 1 (Instance 1) Configuration:**
```bash
echo "<h1>Server 1 - HAProxy Lab</h1>" | sudo tee /var/www/html/index.html
```
**Web Server 2 (Instance 2) Configuration:**
```bash
echo "<h1>Server 2 - HAProxy Lab</h1>" | sudo tee /var/www/html/index.html
```

### **2️⃣ Configure HAProxy Load Balancer**  
On a **third EC2 instance**, I installed **HAProxy**:  
```bash
sudo apt update
sudo apt install haproxy -y
```

I configured `/etc/haproxy/haproxy.cfg`:
```cfg
frontend http_front
    bind *:80
    default_backend web_servers

backend web_servers
    balance roundrobin  # Load balancing method (Round Robin / LeastConn / IP Hash)
    server web1 <WebServer1-IP>:8081 check
    server web2 <WebServer2-IP>:8082 check
```
Then restarted HAProxy:
```bash
sudo systemctl restart haproxy
```

### **3️⃣ Implement Load Balancing Algorithms**  
I modified `haproxy.cfg` to test different algorithms:

#### **🔄 Round Robin (Default)**
```cfg
balance roundrobin
```

#### **📌 IP Hash**
```cfg
balance source
```

#### **⚡ Least Connections**
```cfg
balance leastconn
```

### **4️⃣ Configure Health Checks**
HAProxy ensures only healthy servers receive traffic:
```cfg
server web1 <WebServer1-IP>:8081 check
server web2 <WebServer2-IP>:8082 check
```

---

## **🛠️ Performance Testing**  
**Tool Used:** `h2load` from `nghttp2`  
**Test Parameters:**  
- 100 Requests  
- 10 Concurrent Clients  

I ran:
```bash
h2load -n 100 -c 10 -m 10 http://<HAProxy-IP>
```

### **📊 Results Summary**
| Algorithm         | Req/s   | Mean Request Time | Max Request Time | Min Request Time | TTFB  |
|-----------------|--------|------------------|------------------|------------------|------|
| **Round Robin**     | 838.76  | 11.59ms          | 14.73ms          | 8.20ms           | 12.40ms |
| **IP Hash**         | 957.43  | 9.73ms           | 11.61ms          | 7.31ms           | 10.37ms |
| **Least Connections** | 1506.06 | 5.52ms           | 8.55ms           | 3.65ms           | 5.76ms |

---

## **📌 Conclusion & Recommendation**  
- **Least Connections** is the best-performing algorithm, with the **fastest response times and highest throughput**.  
- **IP Hash** ensures consistency by routing the same client IP to the same backend.  
- **Round Robin** provides equal distribution but **higher latency**.  

For **high-performance applications**, I **recommend Least Connections**.  

---

## **📸 Screenshots & Documentation**  
📂 **Full Implementation Steps:** [`documentation/implementation.md`](./documentation/implementation.md)  
📂 **Testing Report & Results:** [`documentation/testing.md`](./documentation/testing.md)  
📂 **Configuration File:** [`configs/haproxy.cfg`](./configs/haproxy.cfg)  

---

## **💡 Key Learnings**
- **HAProxy efficiently manages load balancing across multiple servers.**  
- **Health checks prevent downtime by excluding unresponsive servers.**  
- **Least Connections** provides the best efficiency for **dynamic traffic environments**.  
