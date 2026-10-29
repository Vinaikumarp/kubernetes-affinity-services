# Kubernetes Node Affinity and Service Configuration

## Project Overview

This project demonstrates how to configure **Node Affinity** in Kubernetes to control Pod scheduling and expose an application using a Kubernetes Service.

Node Affinity helps schedule Pods on specific nodes based on node labels, while a Service provides network access to the application Pods.

## Technologies Used

- Kubernetes
- kubectl
- YAML
- NGINX (sample application)

## Project Objectives

- Understand Kubernetes Node Affinity.
- Schedule Pods based on node labels.
- Create and manage Kubernetes resources using YAML manifests.
- Expose application Pods using a Kubernetes Service.
- Troubleshoot Service selectors and Pod connectivity.

## Project Files

```text
.
├── nodeaffinity_pod.yml
├── nodeaffinity_service.yml
└── README.md
```

## Step 1: Check Cluster Nodes

```bash
kubectl get nodes -o wide
```

## Step 2: Label a Node

Add a label to the node where you want the Pod to be scheduled.

```bash
kubectl label nodes <node-name> app=nginx
```

Verify the label:

```bash
kubectl get nodes --show-labels
```

## Step 3: Configure Node Affinity

In `nodeaffinity_pod.yml`, configure the Pod's `affinity` section to match the node label.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nodeaffinity-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: app
                operator: In
                values:
                  - nginx
```

This configuration requires Kubernetes to schedule the Pod on a node whose `app` label has the value `nginx`.

## Step 4: Create the Pod

```bash
kubectl apply -f nodeaffinity_pod.yml
```

Verify the Pod:

```bash
kubectl get pods -o wide
kubectl describe pod nodeaffinity-pod
```

## Step 5: Configure the Kubernetes Service

Create `nodeaffinity_service.yml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeaffinity-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

**Important:** The Service selector `app: nginx` must match the Pod label `app: nginx`.

## Step 6: Create the Service

```bash
kubectl apply -f nodeaffinity_service.yml
```

Verify the Service:

```bash
kubectl get services
kubectl describe service nodeaffinity-service
kubectl get endpoints nodeaffinity-service
```

## Step 7: Access the Application

Check the NodePort:

```bash
kubectl get service nodeaffinity-service
```

Access the application using:

```text
http://<node-ip>:30080
```

Ensure the node IP is reachable and the required firewall or security-group rules allow port `30080`.

## Step 8: Troubleshooting

### Service selector error

If you receive an error such as:

```text
cannot unmarshal object into Go struct field ServiceSpec.spec.selector of type string
```

Ensure the Service selector uses key-value pairs:

```yaml
selector:
  app: nginx
```

Do not use a nested object under `selector`.

### Pod remains Pending

Check node labels and Pod events:

```bash
kubectl get nodes --show-labels
kubectl describe pod nodeaffinity-pod
```

Ensure at least one eligible node matches the required affinity rules and has sufficient resources.

### Service has no endpoints

Check the Pod labels:

```bash
kubectl get pods --show-labels
```

Confirm that the Service selector matches the labels of the running Pods.

## Cleanup

Delete the resources when the task is complete:

```bash
kubectl delete -f nodeaffinity_service.yml
kubectl delete -f nodeaffinity_pod.yml
```

Remove the node label if it is no longer required:

```bash
kubectl label nodes <node-name> app-
```

## Key Learnings

- Node Affinity controls Pod scheduling based on node labels.
- Kubernetes Services expose applications and route traffic to matching Pods.
- Service selectors must match Pod labels.
- `kubectl describe` and `kubectl get endpoints` are useful troubleshooting commands.

## Conclusion

This task demonstrates how to use Node Affinity to influence Kubernetes Pod placement and a NodePort Service to expose an application. It also covers resource validation and basic troubleshooting of scheduling and networking issues.
