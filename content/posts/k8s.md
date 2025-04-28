---
title: "🧵 Kubernetes Explained from Scratch: Architecture, Commands, and Complete Beginner Guide"
date: 2025-04-27
description: "Everything you need to know about Kubernetes — starting from why it was created, detailed architecture, key components, and almost all useful kubectl commands explained clearly with 5-line explanations. Master Kubernetes step-by-step even if you're seeing it for the first time."
tags: ["kubernetes", "containers", "devops", "cloud", "orchestration"]
---

# 🚀 Kubernetes Explained from Scratch: Architecture, Commands, and Complete Beginner Guide

---

# 🛑 Why Kubernetes?

When companies started adopting container technology (like Docker), they realized that manually managing hundreds or thousands of containers was extremely painful.  
Containers need to be started, monitored, restarted if they crash, scaled up/down based on load, and distributed across servers.  
Without automation, keeping track of containers manually would be nearly impossible at scale.  
Kubernetes solves this by automatically managing containers for deployment, scaling, and recovery.  
It gives a consistent, reliable platform to run containerized applications, regardless of where they are deployed (cloud, on-premises, hybrid).

---

# 🎯 What is Kubernetes?

Kubernetes, also called K8s, is an open-source container orchestration platform developed initially by Google.  
It automates deployment, scaling, and operations of application containers across clusters of hosts.  
You describe your desired state (e.g., 5 replicas of a web server), and Kubernetes maintains that state automatically.  
Kubernetes abstracts away the underlying infrastructure (VMs, physical servers, etc.) so you interact at a higher level.  
It enables zero-downtime deployments, efficient scaling, self-healing of failed applications, and rolling updates.

---

# 🏗️ Kubernetes Detailed Architecture

Understanding Kubernetes architecture is crucial because it reveals how components interact to manage your apps.  
Kubernetes follows a master-worker architecture where the Control Plane (Master Nodes) makes decisions and Worker Nodes run actual applications.  
Each component has a clear responsibility: scheduling, maintaining desired state, health monitoring, etc.  
The architecture enables Kubernetes to be resilient, scalable, and flexible across different types of infrastructure.  
Let’s now break down each component and their roles.

---

## 🔹 1. Control Plane (Master Components)

Control Plane is the brain of Kubernetes. It makes global decisions (like scheduling), and detects/responds to cluster events.

- **kube-apiserver**:  
  The front-end of the Kubernetes Control Plane.  
  All REST API calls (kubectl, dashboards, etc.) interact with it.  
  It validates and processes requests before forwarding to other components.  
  It’s stateless; can be horizontally scaled.

- **etcd**:  
  A lightweight, distributed key-value store.  
  It stores all cluster data — configurations, secrets, service discovery info.  
  Consistency is critical, hence uses the Raft consensus algorithm.  
  Only Control Plane interacts directly with etcd.  
  Backing up etcd regularly is crucial for disaster recovery.

- **kube-scheduler**:  
  Watches for newly created Pods that have no assigned node.  
  It selects the best Node to run the Pod, based on resource requirements and rules.  
  Considers CPU, memory, affinity rules, taints, and tolerations.  
  Pluggable architecture: custom schedulers can be written.  
  Ensures efficient utilization of resources.

- **kube-controller-manager**:  
  Runs controller processes in background to regulate cluster state.  
  Examples: Node controller, Replication controller, Endpoint controller, etc.  
  Each controller watches shared state through kube-apiserver and tries to move current state to desired state.  
  It is a single binary but logically separate controllers.  
  Helps Kubernetes maintain desired behavior automatically.

- **cloud-controller-manager**:  
  Separates cloud-specific control logic from core Kubernetes components.  
  It interacts with cloud provider APIs for things like load balancers, volumes, networking.  
  Essential in cloud deployments like AWS, Azure, GCP.  
  Runs only if the cluster is deployed in a cloud environment.  
  Improves modularity by isolating cloud-dependent code.

---

## 🔹 2. Node Components (Worker Components)

Nodes run the actual application workloads. Each node has several critical components:

- **kubelet**:  
  An agent running on each node.  
  It ensures that containers described in PodSpecs are running and healthy.  
  Communicates with the Control Plane through kube-apiserver.  
  Also reports node status back to the Control Plane.  
  It is responsible for lifecycle management of pods.

- **kube-proxy**:  
  Handles network routing inside the Kubernetes cluster.  
  Maintains network rules on nodes and enables communication to/from pods.  
  Implements load balancing for services across multiple pods.  
  Can use iptables or IPVS under the hood for traffic routing.  
  Ensures network connectivity within the cluster.

- **Container Runtime**:  
  The software responsible for running containers.  
  Kubernetes supports multiple runtimes: Docker, containerd, CRI-O, etc.  
  Kubelet talks to the container runtime via the Container Runtime Interface (CRI).  
  Container runtime pulls container images, starts containers, stops containers.  
  It’s a critical dependency of the Node.

---

## 🔹 3. Add-ons

Additional services that enhance Kubernetes functionality:

- **CoreDNS**:  
  Provides DNS-based service discovery inside Kubernetes.  
  Every service gets an internal DNS entry.  
  Apps can access services via friendly names like `my-service.my-namespace.svc.cluster.local`.  
  CoreDNS dynamically updates DNS records as services change.  
  Crucial for service-to-service communication.

- **Dashboard**:  
  A web-based Kubernetes UI for managing clusters.  
  You can deploy resources, view logs, monitor cluster health visually.  
  Useful for beginners and operational troubleshooting.  
  Needs proper authentication setup to avoid security risks.  
  Can be deployed as a pod inside the cluster.

- **Metrics Server**:  
  Collects resource usage metrics from Kubelets (CPU, memory).  
  Enables features like Horizontal Pod Autoscaling based on usage.  
  It’s a lightweight solution compared to full-fledged Prometheus monitoring.  
  Installed separately into the cluster.  
  Essential for autoscaling workloads dynamically.

---

# 🛠️ Kubernetes Deployment Models

- **On-premises**: Installing Kubernetes yourself on physical or virtual servers.
- **Cloud-managed**: Using EKS (AWS), AKS (Azure), GKE (Google Cloud) services.
- **Hybrid**: Combination of cloud and on-premises clusters.
- **Edge**: Deploying lightweight Kubernetes clusters closer to users/devices.

Each deployment model suits different business needs depending on control, cost, complexity, and scalability.

---

# 🗂️ Common Kubernetes Objects (Resources)

- **Pod**: Smallest deployable unit containing one or more containers.
- **Service**: Abstracts access to a group of Pods; provides stable network endpoint.
- **Deployment**: Manages replicas of Pods and enables updates/rollbacks.
- **ReplicaSet**: Ensures a specified number of pod replicas are running.
- **ConfigMap**: Injects configuration data into pods.
- **Secret**: Stores sensitive information like passwords, tokens securely.
- **Ingress**: Manages external access to services (HTTP/HTTPS routing).

---

# 🧹 Kubernetes Commands  

Let's now go through the **most essential Kubernetes commands**

---

## 🔥 Cluster Information Commands

```bash
kubectl cluster-info
```
Shows details about the Kubernetes Control Plane and core services.  
Helps verify if the cluster is up and reachable.  
Outputs addresses of API server and DNS services.  
First command to check after setting kubeconfig correctly.  
Troubleshoot if cluster access issues appear.

```bash
kubectl get nodes
```
Lists all nodes registered in the cluster.  
Displays node status, roles (master/worker), age, and version.  
Useful to verify if nodes are healthy and ready to run pods.  
If nodes show "NotReady", inspect kubelet/kube-proxy issues.  
Gives an overview of cluster resources.

---

## 📦 Pod Management Commands

```bash
kubectl get pods
```
Lists all pods in the current namespace.  
Shows pod status: Running, Pending, Succeeded, Failed, Unknown.  
Use `-A` or `--all-namespaces` to view across namespaces.  
Essential for monitoring application deployments.  
Identifies which pods need troubleshooting.

```bash
kubectl describe pod <pod-name>
```
Gives detailed information about a specific pod.  
Shows events, conditions, container specs, resource usage, mounted volumes.  
Helpful to debug pod failures (e.g., image pull errors).  
Look at Events section at the bottom carefully.  
More informative than just `get pods`.

```bash
kubectl logs <pod-name>
```
Fetches logs output from a pod's main container.  
Use `-c <container-name>` for multi-container pods.  
Helpful to understand application behavior inside the container.  
Can spot runtime exceptions, crash errors easily.  
Supports following logs with `-f`.

---

## 📋 Deployment Commands

```bash
kubectl create deployment <name> --image=<image-name>
```
Creates a new deployment easily in one line.  
Deploys one or more replica pods running the given container image.  
Automatically sets up ReplicaSet behind the scenes.  
Best suited for initial quick deployments.  
Customize further using manifests later.

```bash
kubectl scale deployment <deployment-name> --replicas=<number>
```
Adjusts the number of pods for an existing deployment.  
Scaling up adds pods; scaling down deletes excess pods gracefully.  
Immediate changes reflected based on cluster resource availability.  
Useful for load-based manual scaling.  
Autoscaling can automate this later.

```bash
kubectl rollout status deployment <deployment-name>
```
Checks rollout progress of a deployment update.  
Ensures new pods are successfully created and old ones terminated.  
Helps monitor rolling updates.  
If stuck, further investigation needed using describe pods.  
Avoids incomplete or unhealthy rollouts.

---

## 🛠️ Service and Networking Commands

```bash
kubectl expose deployment <deployment-name> --type=NodePort --port=<port>
```
Creates a service exposing deployment externally via NodePort.  
Allocates random port on each node mapped to container port.  
Useful for testing from outside the cluster.  
In production, LoadBalancer type or Ingress are preferable.  
Keep NodePorts open carefully in firewall.

```bash
kubectl get services
```
Lists all services running in the cluster.  
Shows service type (ClusterIP, NodePort, LoadBalancer), cluster IP, external IPs.  
Used to check accessibility paths of applications.  
Critical for debugging service communication issues.  
Matches service endpoints to actual pods.

```bash
kubectl port-forward pod/<pod-name> <local-port>:<container-port>
```
Forwards a local port to a port on a pod.  
Enables accessing internal applications without exposing services.  
Ideal for development and debugging.  
Session lasts as long as command is running.  
Quick secure access to private apps.

---

## 🔧 Configuration Commands

```bash
kubectl edit deployment <deployment-name>
```
Opens the live manifest of a deployment in your editor.  
Allows immediate changes to configuration like image version, replicas.  
Saves directly to the cluster on exit.  
Powerful but risky without backups.  
Always validate changes carefully.

```bash
kubectl apply -f <manifest.yaml>
```
Applies configuration defined in a YAML/JSON manifest file.  
Preferred way for managing cluster declaratively.  
Supports creating, updating, patching resources.  
Idempotent — safe to reapply many times.  
Essential for GitOps style workflows.

```bash
kubectl delete pod <pod-name>
```
Deletes a running pod from the cluster.  
Kubernetes automatically recreates it if part of a Deployment.  
Useful to restart unhealthy pods manually.  
Pod deletion triggers graceful shutdown signals.  
Understand controller behavior before deleting pods.

---

## 📥 Namespaces and Context Management

```bash
kubectl get namespaces
```
Lists all namespaces in the cluster.  
Namespaces logically separate cluster resources (like dev, prod).  
Useful for multi-team, multi-project cluster management.  
Built-in namespaces include `default`, `kube-system`, etc.  
Enables better organization and access control.

```bash
kubectl config get-contexts
```
Lists all available kubeconfig contexts.  
Contexts map a cluster, user, and namespace combination.  
Helpful when working across multiple clusters.  
Identifies current active context with an asterisk (*).  
Switch easily between environments (dev, staging, prod).

```bash
kubectl config use-context <context-name>
```
Switches kubectl to a different context.  
Updates which cluster and namespace kubectl commands affect.  
Essential when managing multiple Kubernetes environments.  
Ensures you don't accidentally change wrong clusters.  
Best practice to confirm context before deploying.

---

## 📤 Secrets, ConfigMaps, and Volume Management

```bash
kubectl create secret generic <secret-name> --from-literal=<key>=<value>
```
Creates a basic secret storing sensitive key-value data.  
Secrets encrypt sensitive data like passwords, tokens at rest.  
They are mounted as environment variables or files inside pods.  
Avoid hardcoding sensitive info in manifests.  
Kubernetes handles secret distribution securely.

```bash
kubectl get secrets
```
Lists all secrets available in the current namespace.  
Useful to verify if your secret has been created properly.  
Check types: Opaque, TLS, dockerconfigjson, etc.  
Secrets base64 encode stored data.  
Access restricted to authorized users.

```bash
kubectl create configmap <name> --from-literal=<key>=<value>
```
Creates a ConfigMap from inline literal values.  
Used to inject environment configuration into apps dynamically.  
Decouples configuration from application code.  
Safer and more scalable than baking configs into containers.  
Mounted as files or environment variables.

```bash
kubectl get configmaps
```
Lists all ConfigMaps available in the namespace.  
Displays basic metadata: name, namespace, age.  
Check if app configuration is available before pod creation.  
Helps diagnose issues with missing or outdated config.  
Use describe for detailed key-value listing.

---

## 📈 Autoscaling and Monitoring

```bash
kubectl autoscale deployment <deployment-name> --cpu-percent=<target> --min=<min-pods> --max=<max-pods>
```
Sets up Horizontal Pod Autoscaler (HPA) for a deployment.  
Automatically scales pods up/down based on CPU usage.  
Improves performance during load spikes, saves resources otherwise.  
Metrics Server must be installed for autoscaling to work.  
Tune CPU thresholds according to workload.

```bash
kubectl top nodes
```
Displays CPU and memory usage metrics across all nodes.  
Helps spot overloaded or underutilized nodes quickly.  
Requires Metrics Server running in cluster.  
Crucial for cluster capacity planning.  
Assists in setting correct autoscaling thresholds.

```bash
kubectl top pods
```
Shows CPU and memory usage for each pod.  
Helps identify pods consuming unusual resources.  
Essential for tuning resource requests and limits.  
Resource-efficient clusters reduce cost and improve stability.  
Pod-level view aids micro-level troubleshooting.

---

## 📄 YAML Manifest Example: Deployment

Here’s an example YAML for a simple nginx deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

- Defines a Deployment named `nginx-deployment`.
- Spawns 3 replicas (pods) running `nginx:latest`.
- Each pod exposes port 80 internally.
- Label `app: nginx` used to group pods and services.
- Fully declarative: apply it using `kubectl apply -f`.

---

# 🧠 Kubernetes Key Concepts You Must Understand

## 🌐 Service Discovery

Services expose pods and enable communication. Kubernetes assigns a stable IP and DNS name even if pods behind it change dynamically.  
ClusterIP services are internal-only; NodePort and LoadBalancer expose externally.  
Service discovery enables distributed microservices to interact reliably.  
Without services, IPs would change unpredictably on pod recreation.  
Internal DNS resolves service names to cluster IPs automatically.

---

## 🔁 Rolling Updates and Rollbacks

Deployments support zero-downtime rolling updates of application versions. Kubernetes creates new pods and gradually replaces old ones.  
You can monitor rollout status using `kubectl rollout status`.  
In case of failures, rollbacks restore previous working versions automatically.  
Configuration of rollout strategies like maxUnavailable and maxSurge provides fine control.  
Rolling updates minimize application downtime significantly.

---

## ⚡ Self-Healing

If a container crashes or a node dies, Kubernetes detects the problem automatically.  
Dead pods are rescheduled on healthy nodes without manual intervention.  
Liveness and readiness probes help determine container health accurately.  
This ensures that applications maintain high availability despite failures.  
Self-healing is a core reason Kubernetes is preferred over manual container orchestration.

---

## ⚙️ Desired State Management

In Kubernetes, you declare the desired state through manifests (e.g., 5 replicas running nginx:1.17).  
Kubernetes controllers constantly compare actual state to desired state.  
If discrepancies arise (like only 3 pods running), Kubernetes acts to fix it automatically.  
This declarative model reduces human errors and manual reconciliation.  
It forms the core of Kubernetes’ powerful automation capabilities.

---

# 📋 Real-World Kubernetes Use Cases

- **Microservices Hosting**: Runs thousands of microservices reliably.
- **Machine Learning Pipelines**: Manages distributed training workloads.
- **CI/CD Automation**: Orchestrates build, test, deploy stages.
- **Hybrid Cloud Apps**: Unifies cloud and on-premises deployments.
- **Edge Computing**: Brings computation closer to users in edge locations.

---

# 🏁 Conclusion

Kubernetes is not just another buzzword — it is the gold standard for container orchestration today.  
Its architecture, from the Control Plane to Worker Nodes, is meticulously designed for scalability, reliability, and self-healing.  
With powerful abstractions like Deployments, Services, and ConfigMaps, it allows you to manage even large-scale systems easily.  
The commands and workflows may feel overwhelming at first, but step-by-step exploration makes Kubernetes mastery very achievable.  
If you are serious about cloud-native development or DevOps, understanding Kubernetes is no longer optional — it's essential.

---

# 📚 Further Reading

- [Kubernetes Official Documentation](https://kubernetes.io/docs/home/)
- [Kubernetes Concepts Overview (Kubernetes.io)](https://kubernetes.io/docs/concepts/overview/)
- [Learn Kubernetes Basics (Kubernetes.io)](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
- [Kubernetes Patterns: Design Patterns for Cloud-Native Applications (O’Reilly)](https://www.oreilly.com/library/view/kubernetes-patterns/9781492050285/)
- [Kubernetes Networking Guide (Cilium)](https://docs.cilium.io/en/stable/gettingstarted/k8s-overview/)
- [Kubernetes Best Practices: Blueprints for Building Successful Applications on Kubernetes (O’Reilly)](https://www.oreilly.com/library/view/kubernetes-best-practices/9781492056478/)
- [Katacoda Kubernetes Scenarios (Interactive Labs)](https://www.katacoda.com/courses/kubernetes)
- [Kubernetes the Hard Way (Kelsey Hightower)](https://github.com/kelseyhightower/kubernetes-the-hard-way)

---

💬 *Feel free to connect with me to discuss any project ideas or for collaboration [Connect](https://www.shrivarshapoojary.in/contact)*

---