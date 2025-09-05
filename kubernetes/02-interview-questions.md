### During an IPL final, a new region needs to spin up instantly. How would you pre-warm nodes & scale workloads with zero cold-start impact?

To handle an instant traffic surge for a new region during an IPL final, you need a proactive strategy combining node over-provisioning with intelligent workload placement
This ensures you have warm, available capacity to scale into, eliminating the cold-start delays associated with provisioning new virtual machines from scratch.

The "Over-provisioning with Pause Pods" Technique

The core of this strategy is to trick the Kubernetes Cluster Autoscaler into pre-provisioning nodes before you actually need them. We do this by deploying low-priority "placeholder" pods that reserve space on these nodes.

Step-by-Step Implementation:

1.  Create a Low-Priority PriorityClass
    First, you need to define a PriorityClass with the lowest possible value. Your actual application pods will have a higher, default priority, allowing them to instantly evict these placeholders
```
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: overprovisioning
value: -1
globalDefault: false
description: "Priority class for placeholder pods used for over-provisioning."
```
2. Deploy a Placeholder Deployment
   Next, create a Deployment of "pause" pods. The pause container is a minimal image that does nothing but hold the pod's resources. These pods are assigned the overprovisioning PriorityClass
   Key configurations for this deployment:

   - priorityClassName: overprovisioning: This is crucial. It marks these pods for immediate eviction.
   - Resource Requests: The CPU and memory requests for these pods should mirror the resource needs of your actual application pods. This ensures that the pre-warmed nodes are correctly sized.
   - Replicas: The number of replicas determines how much excess capacity you want to maintain. This can be a static number or dynamically adjusted based on anticipated traffic.
   - Anti-Affinity: Use pod anti-affinity rules to ensure that each placeholder pod lands on a different node, maximizing the number of pre-warmed nodes.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: overprovisioning-placeholders
spec:
  replicas: 10 # 10 nodes will be pre-warmed
  selector:
    matchLabels:
      run: overprovisioning
  template:
    metadata:
      labels:
        run: overprovisioning
    spec:
      priorityClassName: overprovisioning
      containers:
      - name: pause-container
        image: registry.k8s.io/pause
        resources:
          requests:
            cpu: "1"
            memory: "2Gi"
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: run
                operator: In
                values:
                - overprovisioning
            topologyKey: "kubernetes.io/hostname"
```
3. How It Works During the Surge
   1. Pre-Event: The overprovisioning-placeholders Deployment runs, causing the Cluster Autoscaler (ideally an advanced one like Karpenter for speed) to provision 10 new nodes to accommodate them. These nodes are now "warm" and ready.
   2. Traffic Hits: Your application's Horizontal Pod Autoscaler (HPA) or KEDA (Kubernetes Event-driven Autoscaler) detects the incoming traffic surge and starts creating new application pods.
   3. Instant Scheduling: The Kubernetes scheduler sees the new, high-priority application pods. It finds the nodes occupied by the low-priority placeholder pods.
   4. Preemption: The scheduler immediately evicts the placeholder pods from these nodes. This is a near-instantaneous process.
   5. Zero Cold Start: The new application pods are scheduled onto the now-vacant, pre-warmed nodes. They skip the time-consuming node provisioning and initialization steps entirely.
  
4. Additional Optimizations for Zero Impact
  - Pre-Pulled Images: Ensure your application's container image is already present on the pre-warmed nodes. You can achieve this using a DaemonSet that pulls the specific image onto every node in the cluster. This eliminates image pull time, which can be a significant contributor to cold starts
  - Efficient Ingress and Health Checks: Make sure your ingress controller and service mesh are properly configured to rapidly detect the new, healthy pods and route traffic to them. Fast readiness and liveness probes are essential.
  - Predictive Scaling: For a scheduled event like the IPL final, don't rely solely on reactive scaling. A few hours before the match, scale up the number of overprovisioning replicas to build up your warm node pool in anticipation of the traffic spike.

