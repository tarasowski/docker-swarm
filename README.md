# Docker Swarm

Swarm mode

    Note

    Swarm mode is an advanced feature for managing a cluster of Docker daemons.

    Use Swarm mode if you intend to use Swarm as a production runtime environment.

    If you're not planning on deploying with Swarm, **use Docker Compose** instead. If you're developing for a Kubernetes deployment, consider using the integrated Kubernetes feature in Docker Desktop.

Current versions of Docker include Swarm mode for natively managing a cluster of Docker Engines called a swarm. Use the **Docker CLI to create a swarm**, deploy application services to a swarm, and manage swarm behavior.

**Docker Swarm mode** is built into the Docker Engine. Do not confuse Docker Swarm mode with **Docker Classic Swarm**

- which is no longer actively developed.
- Feature highlights
- Cluster management integrated with Docker Engine

Use the Docker Engine CLI to create a swarm of Docker Engines where you can deploy application services. You don't need additional orchestration software to create or manage a swarm.

## Decentralized design

Instead of handling differentiation between node roles at deployment time, the Docker Engine handles any specialization at runtime. You can deploy both kinds of nodes, managers and workers, using the Docker Engine. This means you can build an entire swarm from a single disk image.

## Declarative service model

Docker Engine uses a declarative approach to let you **define the desired state** of the various services in your application stack. For example, you might describe an application comprised of a web front end service with message queueing services and a database backend.

## Scaling

For each service, **you can declare the number of tasks** you want to run. When you scale up or down, the swarm manager automatically adapts by adding or removing tasks to maintain the desired state.

# Difference Between a Service and a Task in Docker Swarm

In Docker Swarm, **services** and **tasks** are fundamental concepts that help manage and orchestrate containers in a cluster. Here's the difference between them:

---

## **Service**
- A **service** is a definition of the desired state of a containerized application in a Swarm cluster.
- It specifies:
  - The container image to use.
  - The number of replicas (tasks) to run.
  - Network and storage configurations.
  - Port mappings.
  - Restart policies.
  - Update and rollback strategies.
- A service is a declarative way to describe how your application should run in the Swarm.
- **Example**: You define a service to run 5 replicas of a web server using the `nginx` image.

---

## **Task**
- A **task** is a single instance of a service running on a node in the Swarm cluster.
- Each task corresponds to a container that is scheduled and managed by the Swarm manager.
- If a service is defined to have multiple replicas, each replica is a separate task.
- Tasks are dynamically created, started, stopped, or removed by the Swarm manager to maintain the desired state of the service.
- **Example**: If a service is defined to run 5 replicas, there will be 5 tasks, each running a container on one of the nodes in the Swarm.

---

## **Key Differences**
| **Aspect**          | **Service**                                      | **Task**                                      |
|----------------------|--------------------------------------------------|-----------------------------------------------|
| **Definition**       | Describes the desired state of the application. | Represents a single instance of a container. |
| **Scope**            | Cluster-wide.                                    | Node-specific.                                |
| **Replicas**         | Defines how many tasks (containers) to run.     | A single container instance.                 |
| **Management**       | Managed by the Swarm manager.                   | Managed by the Swarm manager as part of a service. |
| **Example**          | A service to run 3 replicas of a web app.       | Each of the 3 running containers is a task.  |

---

## **Example in Practice**
1. You create a service in Docker Swarm:
   ```bash
   docker service create --name web --replicas 3 nginx

    This defines a service named web that should run 3 replicas of the nginx container.

    The Swarm manager schedules 3 tasks (one for each replica) across the nodes in the cluster. Each task runs an nginx container.

    If a node fails, the Swarm manager reschedules the affected task(s) on other nodes to maintain the desired state of 3 replicas.

   ``` 

## Desired state reconciliation

The swarm manager node constantly monitors the cluster state and reconciles any differences between **the actual state and your expressed desired state**. **For example, if you set up a service to run 10 replicas of a container, and a worker machine hosting two of those replicas crashes, the manager creates two new replicas to replace the replicas that crashed.** The swarm manager assigns the new replicas to workers that are running and available.

## Multi-host networking

You can specify an overlay network for your services. The swarm manager automatically assigns addresses to the containers on the overlay network when it initializes or updates the application.

## Service discovery

Swarm manager nodes assign each service in the swarm a **unique DNS name** and **load balance running containers**. You can **query every container running in the swarm through a DNS server** embedded in the swarm.
Load balancing

You can **expose the ports for services to an external load balancer**. Internally, the swarm lets you specify how to distribute service containers between nodes.

## Secure by default

Each node in the swarm enforces TLS mutual authentication and encryption to **secure communications between itself and all other nodes**. You have the option to use self-signed root certificates or certificates from a custom root CA.

## Rolling updates

At rollout time you can apply service updates to nodes incrementally. The swarm manager lets you control the delay between service deployment to different sets of nodes. **If anything goes wrong, you can roll back to a previous version of the service.**

---

# Summary: Administer and Maintain a Swarm of Docker Engines

## **Key Concepts**
- **Manager Nodes**: Central to managing the swarm and storing its state. They use the **Raft Consensus Algorithm** to maintain consistency.
- **Worker Nodes**: Execute tasks assigned by manager nodes.
- **Quorum**: A majority of manager nodes must agree on updates to the swarm state. Losing quorum prevents management tasks.

---

## **Manager Nodes**
- **Raft Consensus**: Ensures consistency across manager nodes. More managers increase fault tolerance but reduce write performance due to increased network traffic.
- **Quorum Maintenance**:
  - Always maintain an **odd number of managers** for fault tolerance.
  - Example: With 3 managers, you can lose 1; with 5, you can lose 2.
  - Losing quorum halts management tasks but does not affect running tasks on worker nodes.
- **Static IP Address**: Manager nodes should use a fixed IP address (`--advertise-addr`) to avoid instability during reboots.

In Docker Swarm, **quorum** refers to the minimum number of **manager nodes** that must be available and in agreement to make decisions and maintain the swarm's state. It is a critical concept for ensuring the consistency and fault tolerance of the swarm.

---

### **How Quorum Works**
1. **Raft Consensus Algorithm**:
   - Docker Swarm uses the **Raft Consensus Algorithm** to manage the swarm state.
   - For any update (e.g., adding/removing nodes, updating services), a **majority (quorum)** of manager nodes must agree on the change.

2. **Majority Calculation**:
   - Quorum is calculated as a **majority** of the total number of manager nodes.
   - Formula: `Quorum = (Number of Managers / 2) + 1` (rounded down).

   | **Number of Managers** | **Quorum (Majority)** | **Fault Tolerance** |
   |-------------------------|-----------------------|---------------------|
   | 1                       | 1                     | 0 (no fault tolerance) |
   | 2                       | 2                     | 0 (no fault tolerance) |
   | 3                       | 2                     | 1 (can lose 1 manager) |
   | 4                       | 3                     | 1 (can lose 1 manager) |
   | 5                       | 3                     | 2 (can lose 2 managers) |

3. **Why Quorum Matters**:
   - Ensures that the swarm state is consistent across all manager nodes.
   - Prevents "split-brain" scenarios where different managers have conflicting views of the swarm state.

---

### **What Happens If Quorum is Lost?**
- If the number of available manager nodes falls below the quorum:
  - The swarm **cannot process updates** (e.g., adding/removing nodes, updating services).
  - Existing tasks on worker nodes **continue to run**, but no new tasks can be scheduled.
  - The swarm is effectively **unmanageable** until quorum is restored.

---

### **Example Scenarios**
1. **3 Manager Nodes**:
   - Quorum = 2.
   - If 1 manager is lost, the swarm continues to function normally.
   - If 2 managers are lost, the swarm loses quorum and becomes unmanageable.

2. **5 Manager Nodes**:
   - Quorum = 3.
   - If 2 managers are lost, the swarm continues to function.
   - If 3 managers are lost, the swarm loses quorum.

---

### **Best Practices for Maintaining Quorum**
1. **Use an Odd Number of Managers**:
   - Always deploy an **odd number of manager nodes** (e.g., 3, 5, 7).
   - This ensures that a majority can always be achieved, even if some nodes fail.

2. **Distribute Managers Across Availability Zones**:
   - Spread manager nodes across different physical or cloud availability zones to avoid single points of failure.

3. **Monitor Manager Health**:
   - Regularly check the status of manager nodes using `docker node ls` and `docker node inspect`.

4. **Replace Lost Managers Promptly**:
   - If a manager node is lost, replace it as soon as possible to maintain fault tolerance.

5. **Avoid Scaling Down to a Single Manager**:
   - A single manager has no fault tolerance. If it fails, the swarm becomes unmanageable.

---

### **Recovering from Quorum Loss**
If the swarm loses quorum:
1. **Bring Failed Managers Back Online**:
   - Restart the failed manager nodes if possible.
2. **Force a New Cluster**:
   - On the remaining manager node, run:
     ```bash
     docker swarm init --force-new-cluster
     ```
   - This restores quorum with a single manager, but you should add more managers afterward.

---

### **Summary**
- **Quorum** is the minimum number of manager nodes required to make decisions and maintain the swarm state.
- It ensures consistency and fault tolerance in Docker Swarm.
- Always maintain an **odd number of managers** and monitor their health to avoid losing quorum.


---

## **Fault Tolerance**
- **Manager Node Distribution**:
  - Distribute managers across **3 availability zones** for optimal fault tolerance.
  - Example: For 5 managers, distribute as 2-2-1 across zones.
- **Adding Managers**:
  - Add manager nodes to increase fault tolerance.
  - Avoid scaling down to a single manager, as it risks swarm unavailability.

---

## **Manager-Only Nodes**
- By default, manager nodes also act as workers. To prevent resource starvation:
  - Drain manager nodes: `docker node update --availability drain NODE`.
  - This ensures managers focus on swarm management tasks.

---

## **Worker Nodes**
- Add worker nodes to balance load across the swarm.
- Tasks are distributed based on node resources (e.g., CPU, memory).
- Worker nodes with insufficient resources cannot run certain tasks.

---

## **Monitoring Swarm Health**
- Use `docker node inspect <node-id>` to check:
  - Manager reachability: `--format "{{ .ManagerStatus.Reachability }}"`.
  - Worker status: `--format "{{ .Status.State }}"`.
- Use `docker node ls` for an overview of swarm nodes and their statuses.

---

## **Troubleshooting**
- **Unreachable Managers**:
  - Restart the daemon or reboot the machine.
  - If unresolved, add a new manager or promote a worker.
- **Forcibly Remove a Node**:
  - Use `docker node rm --force <node>` for unreachable or compromised nodes.
  - Demote manager nodes before removal.

---

## **Backup and Recovery**
- **Backup**:
  - Stop Docker on a manager node.
  - Backup the `/var/lib/docker/swarm` directory.
  - Restart Docker after backup.
- **Restore**:
  - Restore the `/var/lib/docker/swarm` directory on a new node.
  - Reinitialize the swarm: `docker swarm init --force-new-cluster`.
  - Add managers and workers to restore full functionality.

---

## **Recovering from Quorum Loss**
- If quorum is lost:
  - Bring failed manager nodes back online if possible.
  - Use `docker swarm init --force-new-cluster` on a remaining manager to restore quorum.
  - Re-add or promote nodes to rebuild the manager set.

---

## **Rebalancing Tasks**
- Swarm does not automatically rebalance tasks to avoid disrupting running services.
- Force rebalancing with `docker service update --force` (restarts tasks).
- Alternatively, temporarily scale services up and down to redistribute tasks.

---

## **Best Practices**
- Maintain an odd number of managers for fault tolerance.
- Distribute managers across availability zones.
- Use static IPs for manager nodes.
- Regularly back up the swarm state.
- Avoid scaling down to a single manager.

# Getting Started With Swarm Mode

[Tutorial Nginx + Visualizer](https://github.com/tarasowski/docker-swarm/blob/main/Intro%2BTutorial.md)

[5 Node Cluster Tutorial](https://dockerlabs.collabnix.com/intermediate/workshop/getting-started-with-swarm.html)

[Official Docker Tutorial](https://docs.docker.com/engine/swarm/swarm-tutorial/)
