Here's a Docker Swarm example for deploying an Nginx service only on a worker node, with a visualizer for monitoring the cluster.

---

### **Steps:**
1. **Launch two EC2 instances** (one for the manager, one for the worker).
2. **Install Docker on both instances**.
3. **Initialize Docker Swarm** on the manager.
4. **Join the worker node** to the Swarm.
5. **Deploy Nginx on the worker node**.
6. **Deploy a Swarm visualizer** on the manager.

### **Required Ports for Docker Swarm on EC2**
You need to configure **security groups** in AWS to allow communication between the manager and worker nodes.

---

## **1. Manager Node:**
| Port | Protocol | Purpose |
|------|---------|---------|
| 22   | TCP     | SSH access |
| 2377 | TCP     | Swarm manager communication (cluster management) |
| 7946 | TCP/UDP | Node discovery |
| 4789 | UDP     | VXLAN-based overlay networking |
| 8081 | TCP     | Swarm Visualizer UI (optional) |

---

## **2. Worker Node:**
| Port | Protocol | Purpose |
|------|---------|---------|
| 22   | TCP     | SSH access |
| 7946 | TCP/UDP | Node discovery |
| 4789 | UDP     | VXLAN-based overlay networking |
| 8080 | TCP     | Nginx service (external access) |

---

## **3. Additional Considerations**
- **If using AWS Load Balancer**, ensure the public-facing ports (e.g., 8080 for Nginx) are open.
- **For inter-node communication**, security groups should allow traffic between the manager and workers on **2377, 7946, and 4789**.

---

## **Security Group Example**
### **For Manager Node:**
- **Inbound Rules:**
  - Allow **SSH (22/TCP)** from your IP.
  - Allow **Swarm traffic (2377/TCP, 7946/TCP/UDP, 4789/UDP)** from worker nodes.
  - Allow **Visualizer UI (8081/TCP)** from your IP.

### **For Worker Node:**
- **Inbound Rules:**
  - Allow **SSH (22/TCP)** from your IP.
  - Allow **Swarm traffic (7946/TCP/UDP, 4789/UDP)** from the manager.
  - Allow **Nginx (8080/TCP)** from the internet (or restrict as needed).

---

## **Testing Connectivity**
After setting up security groups, test connectivity between nodes:

```sh
nc -zv <WORKER_PRIVATE_IP> 2377  # From Manager to Worker
nc -zv <MANAGER_PRIVATE_IP> 2377 # From Worker to Manager
```

---

### **1. Install Docker on Both EC2 Instances**
Run this on **both** the manager and worker nodes:

```sh
sudo apt update -y && sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```

---

### **2. Initialize Docker Swarm on Manager Node**
On the **manager node**, run:

```sh
docker swarm init --advertise-addr <MANAGER_PRIVATE_IP>
```

This will return a command like:

```sh
docker swarm join --token SWMTKN-1-xxxxxx <MANAGER_PRIVATE_IP>:2377
```

---

### **3. Join the Worker Node**
On the **worker node**, run the join command returned in the previous step:

```sh
docker swarm join --token SWMTKN-1-xxxxxx <MANAGER_PRIVATE_IP>:2377
```

Verify the worker joined successfully by running this on the manager:

```sh
docker node ls
```

---

### **4. Deploy Nginx Only on the Worker Node**
Run this on the **manager node**:

```sh
docker service create \
  --name nginx-worker \
  --constraint 'node.role == worker' \
  --publish 8080:80 \
  nginx
```

This ensures Nginx is only deployed on a **worker node**.

Verify the deployment:

```sh
docker service ls
docker ps
```

---

### **5. Deploy Swarm Visualizer on the Manager**
Run this on the **manager node**:

```sh
docker service create \
  --name swarm-visualizer \
  --constraint 'node.role == manager' \
  --publish 8081:8080 \
  --mount type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
  dockersamples/visualizer
```

---

### **6. Access the Services**
- **Nginx on Worker** → `http://<ANY_NODE_PUBLIC_IP>:8080`
- **Visualizer on Manager** → `http://<MANAGER_PUBLIC_IP>:8081`

---

This setup ensures:
✅ Nginx runs **only** on the worker node.  
✅ The **Visualizer** runs **only** on the manager node.  
✅ You can monitor the swarm with **Visualizer**.
