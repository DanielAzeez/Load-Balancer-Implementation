### **📌 `testing.md`** (Performance Testing and Results Analysis)  

```md
# HAProxy Load Balancer Testing & Performance Analysis

## **Overview**
After setting up HAProxy as a load balancer, I performed **load testing** to analyze how different load balancing algorithms handle traffic distribution and request performance. The three algorithms tested were:
1. **Round Robin**
2. **IP Hash**
3. **Least Connections**

Testing was conducted using **h2load**, a benchmarking tool from `nghttp2`, with 100 HTTP requests distributed across backend servers.  

---

## **Test Setup**
### **Environment**
- **Load Testing Tool:** `h2load` (from `nghttp2` package)
- **Load Balancer:** HAProxy
- **Backend Servers:** 2 Nginx servers
- **Test Parameters:**
  - **Number of Requests:** 100
  - **Concurrent Streams:** 10
  - **Clients:** 10  

### **Installation of h2load**
I installed `h2load` on the HAProxy server:
```bash
sudo apt update
sudo apt install nghttp2-client -y
```

### **Running the Tests**
I used the following command to test HAProxy's load balancing:
```bash
h2load -n 100 -c 10 -m 10 http://<Load-Balancer-IP>
```
This simulates **100 total requests** from **10 clients**, with each client maintaining **10 concurrent streams**.

---

## **Performance Metrics Evaluated**
- **Requests Per Second (req/s)** – Measures request throughput
- **Time for Request** – Measures the latency of requests
- **Time to Connect** – Measures connection time
- **Time to First Byte (TTFB)** – Time taken before the first byte of the response is received
- **Standard Deviation (SD)** – Measures performance stability  

---

## **Results Analysis**

### **1️⃣ Round Robin**
- **Requests Per Second:** ~838.76 req/s
- **Time for Request:** Mean: 11.59ms, Max: 14.73ms, Min: 8.20ms
- **Time to Connect:** Mean: 588µs
- **Time to First Byte:** Mean: 12.40ms
- **Observation:**  
  - Round Robin evenly distributed requests between the two backend servers.
  - The response time fluctuated due to varying server loads.
  - Some requests experienced **higher latency** than others.

### **2️⃣ IP Hash**
- **Requests Per Second:** ~957.43 req/s
- **Time for Request:** Mean: 9.73ms, Max: 11.61ms, Min: 7.31ms
- **Time to Connect:** Mean: 492µs
- **Time to First Byte:** Mean: 10.37ms
- **Observation:**  
  - Requests from the same client IP were routed to the same backend.
  - This method improved request consistency compared to Round Robin.
  - It performed **better than Round Robin** but was **less efficient than Least Connections**.

### **3️⃣ Least Connections**
- **Requests Per Second:** ~1506.06 req/s
- **Time for Request:** Mean: 5.52ms, Max: 8.55ms, Min: 3.65ms
- **Time to Connect:** Mean: 415µs
- **Time to First Byte:** Mean: 5.76ms
- **Observation:**  
  - Requests were assigned to the backend server **with the least active connections**.
  - This method **significantly reduced latency**.
  - It achieved the **highest throughput and lowest request times**, making it the most efficient.

---

## **Comparison Table**
| Algorithm         | Req/s   | Mean Request Time | Max Request Time | Min Request Time | TTFB  |
|-----------------|--------|------------------|------------------|------------------|------|
| Round Robin     | 838.76  | 11.59ms          | 14.73ms          | 8.20ms           | 12.40ms |
| IP Hash         | 957.43  | 9.73ms           | 11.61ms          | 7.31ms           | 10.37ms |
| Least Connections | 1506.06 | 5.52ms           | 8.55ms           | 3.65ms           | 5.76ms |

---

## **Conclusion**
1. **Least Connections** outperformed the other two methods with the lowest latency and highest throughput.
2. **IP Hash** provided better performance than Round Robin but wasn't as optimized as Least Connections.
3. **Round Robin** evenly distributed traffic but led to **higher latency due to unoptimized server selection**.

### **Final Recommendation**
For **high-performance** load balancing, **Least Connections** is the best choice, as it dynamically optimizes server load and provides **faster response times**.
