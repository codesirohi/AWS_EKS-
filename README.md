# AWS_EKS


# AWS EKS Components and Interaction

## Components and Their Interaction

### 1. Pods and Microservice Containers
- **Single Microservice per Pod**: Each pod contains a single container running a microservice.
- **Sidecar Pattern**: Each pod contains multiple containers, usually a main container running the microservice and a sidecar container for related tasks (e.g., logging, monitoring).

### 2. EC2 Instances and Node Groups
- **Worker Nodes**: EC2 instances run as worker nodes, managed by EKS. They host the pods.
- **Node Groups**: Collections of EC2 instances managed as a group. Node groups can be auto-scaling groups (ASGs) which automatically manage the scaling of the nodes.

### 3. Master Node (Control Plane)
- **EKS Control Plane**: AWS manages the Kubernetes master nodes. They handle API requests, scheduling, and overall cluster management.

### 4. Scaling
- **EC2 Instance Scaling**: Managed by auto-scaling groups based on metrics (e.g., CPU usage, custom metrics).
- **Pod Scaling**: Managed by Kubernetes via Horizontal Pod Autoscaler (HPA), which scales pods based on metrics like CPU, memory usage, or custom metrics from Prometheus.

### 5. Services and Load Balancers
- **Service Types**:
  - **ClusterIP**: Default service type; exposes the service on a cluster-internal IP.
  - **NodePort**: Exposes the service on each node’s IP at a static port.
  - **LoadBalancer**: Exposes the service externally using a cloud provider’s load balancer (e.g., AWS ELB).
  - **Ingress**: Manages external access to services, typically HTTP.

## Detailed Interactions and Scaling

### Scaling Triggers and Mechanisms

#### 1. EC2 Instances Scaling
- **Auto Scaling Groups (ASG)**: Scale EC2 instances up or down based on predefined metrics (e.g., average CPU usage, custom CloudWatch alarms).
- **Cluster Autoscaler**: Adjusts the number of nodes in a cluster. When there are insufficient resources for pods, it increases the node count, and decreases it when nodes are underutilized.

#### 2. Pod Scaling
- **Horizontal Pod Autoscaler (HPA)**: Scales the number of pod replicas based on metrics like CPU, memory usage, or custom metrics.
- **Vertical Pod Autoscaler (VPA)**: Adjusts the resource requests and limits of containers in pods.

### Service Discovery and Load Balancing

#### 1. Service Discovery
- **Kubernetes DNS**: Automatically assigns DNS names to services, enabling other services to discover them via DNS queries.

#### 2. Load Balancing
- **External Load Balancer**: Created by a `LoadBalancer` type service. AWS ELB distributes incoming traffic among pods.
- **Internal Load Balancer**: Managed by Kubernetes, using kube-proxy to route internal traffic among pods.

## Deployment

### 1. Deployment Tools
- **kubectl**: CLI tool for interacting with Kubernetes clusters. Used to deploy, manage, and inspect resources.
- **Terraform**: Infrastructure as Code (IaC) tool for defining and provisioning infrastructure. Can manage EKS clusters and Kubernetes resources.

```bash
# Using kubectl to deploy
kubectl apply -f deployment.yaml

# Using Terraform
terraform init
terraform apply
```

## Traffic Handling in AWS EKS

### Scenario 1: External Traffic to Microservice A

#### Steps:
1. **External Request**:
   - An external client sends a request to Microservice A.

2. **Load Balancer**:
   - The request hits an AWS Elastic Load Balancer (ELB) created by a Kubernetes `LoadBalancer` type service.
   - The ELB distributes the incoming traffic to the worker nodes that have pods of Microservice A running.

3. **Kube-Proxy**:
   - The worker node receiving the request has a `kube-proxy` running.
   - `kube-proxy` uses iptables or IPVS to route the request to one of the pods of Microservice A running on the node.

4. **Pod Selection**:
   - The specific pod of Microservice A is selected based on a round-robin algorithm or another load balancing strategy configured in `kube-proxy`.
   - The request is processed by the selected pod of Microservice A.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: microservice-a
spec:
  type: LoadBalancer
  selector:
    app: microservice-a
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```


### Scenario 2: Internal Traffic from Microservice A to Microservice B

#### Steps:

1. **Request from Microservice A**:
   - After processing the initial request, Microservice A needs to communicate with Microservice B to complete the transaction.

2. **Service Discovery**:
   - Microservice A uses the Kubernetes DNS to discover the service name of Microservice B.
   - The DNS name for Microservice B might be something like `microservice-b.default.svc.cluster.local`.

3. **ClusterIP Service**:
   - The request from Microservice A is sent to the ClusterIP of Microservice B.
   - The ClusterIP is a stable internal IP address that Kubernetes assigns to the service.

4. **Kube-Proxy**:
   - `kube-proxy` on the worker node hosting the pod of Microservice A routes the request to one of the pods of Microservice B.

5. **Pod Selection**:
   - The specific pod of Microservice B is selected based on the same load balancing strategy (e.g., round-robin).
   - The request is processed by the selected pod of Microservice B.

```yaml

apiVersion: v1
kind: Service
metadata:
  name: microservice-b
spec:
  type: ClusterIP
  selector:
    app: microservice-b
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```
