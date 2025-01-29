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


