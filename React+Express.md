## **1. Containerize React and Express**
Ensure both the React and Express apps have `Dockerfile`s.

### **React App (Frontend) - `Dockerfile`**
```dockerfile
# Build stage
FROM node:18 as build
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Set for API_URL in React

```js
const API_URL = process.env.REACT_APP_API_URL || "http://localhost:5000";

fetch(`${API_URL}/api/data`)
  .then((response) => response.json())
  .then((data) => console.log(data));
```

---

### **Express Server (Backend) - `Dockerfile`**
```dockerfile
FROM node:18
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY . .
EXPOSE 5000
CMD ["node", "server.js"]
```

---

## **2. Initialize Docker Swarm**
On your **Swarm Manager**, run:

```sh
docker swarm init --advertise-addr <MANAGER_PRIVATE_IP>
```

Join worker nodes:

```sh
docker swarm join --token SWMTKN-1-xxxxxx <MANAGER_PRIVATE_IP>:2377
```

Verify the nodes:

```sh
docker node ls
```

---

## **3. Create a `docker-compose.yml` for Swarm**
Create a `docker-compose.yml` file for your stack.

**IMPORTANT:** You need to create a local docker registry on a manager node [create local repository](https://github.com/tarasowski/docker-swarm/blob/main/Docker%2BRegistry.md)

```yaml
version: "3.8"

services:
  frontend:
    image: manager-ip:5000/my-react-app:latest
    deploy:
      replicas: 3  # Runs 3 instances for fault tolerance
      restart_policy:
        condition: on-failure
    networks:
      - app-network
    ports:
      - "80:80"
     environment:
      - REACT_APP_API_URL=http://backend:5000  # Swarm Service Discovery
  
  backend:
    image: manager-ip:5000/my-express-server:latest
    deploy:
      replicas: 3  # Runs 3 instances for fault tolerance
      restart_policy:
        condition: on-failure
    networks:
      - app-network
    environment:
      - NODE_ENV=production
    ports:
      - "5000:5000"

networks:
  app-network:
    driver: overlay
```

This configuration:

✅ **Replicates** frontend & backend (3 instances each)  
✅ **Automatically restarts** failed containers  
✅ **Uses Swarm’s built-in load balancing**  
✅ **Communicates via an overlay network**  (An overlay network is a software-defined network that allows containers running on different nodes in a Swarm cluster to communicate securely as if they were on the same physical network.)

---

## **4. Deploy the Stack**
Run this on the **Swarm Manager**:

```sh
docker stack deploy -c docker-compose.yml my-app
```

Check if the services are running:

```sh
docker service ls
```

---

## **5. Load Balancing (Built-in)**
Docker Swarm automatically load balances across nodes. You can access:

- **Frontend (React)** → `http://<ANY_NODE_PUBLIC_IP>:80`
- **Backend (Express API)** → `http://<ANY_NODE_PUBLIC_IP>:5000`

Swarm will distribute requests among all replicas.

---

## **6. Scaling & Rolling Updates**
Increase backend replicas:

```sh
docker service scale my-app_backend=5
```

---

## **7. Health Checks (Optional)**
You can add health checks for better monitoring.

### **Express Health Check**
Modify `docker-compose.yml`:

```yaml
  backend:
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
      healthcheck:
        test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
        interval: 30s
        retries: 3
```

In `server.js` (Express):

```js
app.get('/health', (req, res) => {
  res.status(200).send("OK");
});
```

---

### **Testing**
Run:

```sh
docker stack ps my-app
```

Check logs:

```sh
docker service logs my-app_backend
```

Check cluster nodes:

```sh
docker node ls
```

---

### **Final Architecture (Without Load Balancer & DB)**
- **Frontend (React)** → Replicated (3 instances)
- **Backend (Express)** → Replicated (3 instances)
- **Swarm Network** → Built-in load balancing
- **No External Load Balancer**
- **No Database**

---

This setup ensures **high availability**, **auto-healing**, and **fault tolerance** for your React + Express app **without Traefik or a database**.

Let me know if you need **Terraform automation**! 🚀
