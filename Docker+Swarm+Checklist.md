# Docker Swarm Checklist

## Docker Swarm Commands and Objects

| Command / Object | Description | Example |
|------------------|-------------|---------|
| `docker swarm init` | Initializes a new Docker Swarm cluster on the current node. | `docker swarm init --advertise-addr 192.168.1.100` |
| `docker swarm join` | Joins a node to an existing Swarm as a worker or manager. | `docker swarm join --token <token> 192.168.1.100:2377` |
| `docker swarm leave` | Removes the node from the Swarm cluster. | `docker swarm leave --force` |
| `docker node ls` | Lists all nodes in the Swarm. | `docker node ls` |
| `docker node rm` | Removes a node from the cluster. | `docker node rm node1` |
| `docker service create` | Creates a new service in the Swarm. | `docker service create --name web -p 80:80 nginx` |
| `docker service ls` | Lists all services in the Swarm. | `docker service ls` |
| `docker service rm` | Removes a service from the Swarm. | `docker service rm web` |
| `docker service scale` | Scales a service to the specified number of replicas. | `docker service scale web=5` |
| `docker service ps` | Lists tasks for a service. | `docker service ps web` |
| `docker service inspect` | Provides detailed information about a service. | `docker service inspect web` |
| `docker stack deploy` | Deploys a stack using a Compose file. | `docker stack deploy -c docker-compose.yml my_stack` |
| `docker stack ls` | Lists all stacks in the Swarm. | `docker stack ls` |
| `docker stack rm` | Removes a stack from the Swarm. | `docker stack rm my_stack` |
| `docker network create` | Creates an overlay network for services. | `docker network create -d overlay my_network` |
| `docker network ls` | Lists all networks in the Swarm. | `docker network ls` |
| `docker config create` | Creates a configuration object for Swarm services. | `echo "config-data" | docker config create my_config -` |
| `docker config ls` | Lists all stored configurations. | `docker config ls` |
| `docker secret create` | Creates a secret to be used in Swarm services. | `echo "secret-data" | docker secret create my_secret -` |
| `docker secret ls` | Lists all secrets in the Swarm. | `docker secret ls` |
| `docker stack ps` | Lists tasks for a stack. | `docker stack ps my_stack` |

## Additional Notes
- Ensure the manager node is accessible by worker nodes.
- Use `--advertise-addr` to specify the IP address of the manager node.
- Worker nodes cannot execute management commands.
- Use `docker service update` to modify existing services.
- Swarm uses overlay networks to enable service communication across nodes.
- Use `docker stack deploy` for multi-service deployments with Docker Compose.

This checklist provides an overview of essential Docker Swarm commands and concepts for managing services, nodes, stacks, and networks.

Certainly! Here’s a breakdown of Docker Swarm objects:

### 1. **Nodes**
   - A **node** is an individual instance of the Docker Engine participating in a Swarm cluster.
   - Nodes can be either:
     - **Manager Nodes** – Manage the cluster, schedule tasks, and maintain the desired state.
     - **Worker Nodes** – Execute containers as per instructions from the manager.
   - Example: 
     ```sh
     docker node ls   # Lists all nodes in the Swarm
     ```

### 2. **Services**
   - A **service** is the definition of how a container should run inside a Swarm.
   - Services define **desired state** (e.g., how many replicas should run, network settings, etc.).
   - Example:
     ```sh
     docker service create --name web -p 80:80 nginx
     ```

### 3. **Stacks**
   - A **stack** is a collection of services defined using a Docker Compose file.
   - Used for deploying multi-container applications.
   - Example:
     ```sh
     docker stack deploy -c docker-compose.yml my_stack
     ```

### 4. **Tasks**
   - A **task** represents a running container assigned to a node.
   - When a service is created, Swarm schedules tasks and assigns them to nodes.
   - Tasks are immutable – if a service is updated, new tasks are created while old ones are removed.
   - Example:
     ```sh
     docker service ps web   # Lists tasks for the 'web' service
     ```

### 5. **Replicas**
   - **Replicas** define the number of container instances a service should maintain.
   - Swarm ensures that the desired number of replicas are always running.
   - Example:
     ```sh
     docker service scale web=5  # Scale the web service to 5 replicas
     ```

Each of these objects plays a crucial role in Docker Swarm, ensuring high availability, scalability, and service orchestration. Let me know if you need further details! 🚀
