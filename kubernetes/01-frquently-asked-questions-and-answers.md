### What’s the Difference Between Deployment vs StatefulSet?

<img width="1028" height="319" alt="image" src="https://github.com/user-attachments/assets/e1efc5e0-b5d3-4a47-8f68-496c20be8e6e" />

### How to Debug a CrashLoopBackOff?

Step-by-step approach:

  Check logs of the previous pod:
      kubectl logs <pod-name> --previous

  Check describe output for reasons:
      kubectl describe pod <pod-name>
      Look for Events: OOMKilled, FailedMount, ImagePullBackOff, etc.

  Check readiness/liveness probes:
    Failing probes may restart the container.
    
  Verify Init Containers:
    If failing, the main container won’t start.
    
  Check CPU/memory limits:
    Overly restrictive limits → crashes

Pro Tip: Use kubectl get events --sort-by=.metadata.creationTimestamp for latest failures.

### How Do You Implement Blue-Green or Canary Deployments?

Blue-Green Deployment:

  -  Deploy new version (green) alongside existing (blue)
  -  Use a switch (like LoadBalancer, Ingress) to redirect all traffic
  -  Rollback is just switching traffic back

Tools:

  -  Argo Rollouts
  -  Service selectors
  -  Ingress rules

Canary Deployment:

  -  Release new version to small traffic % (e.g., 10%)
  -  Gradually increase as health is verified
  -  Automated with Ingress or Service Mesh (Istio)

Example Using Istio:
```
- route:
  - destination:
      host: v1
    weight: 90
  - destination:
      host: v2
    weight: 10
```
### How Would You Isolate Workloads in a Multi-Tenant Cluster?

Techniques:

  Namespaces:
  
  - Separate teams/projects/apps

  NetworkPolicies:

  - Control inter-namespace communication
  
  ResourceQuotas & Limits:
  - Prevent noisy neighbors from consuming all resources
  
  RBAC:
  - Fine-grained access control per tenant
  
  PodSecurityPolicies / OPA Gatekeeper:
  - Enforce security/compliance policies

Pro Tip: Combine all of the above for tenant-level sandboxing.

### When to Use Taints/Tolerations vs Node Affinity?

<img width="938" height="231" alt="image" src="https://github.com/user-attachments/assets/b16aa8f3-ea58-49ff-b3b9-079f49f480d5" />

Example:

  -  Use taint on GPU node: dedicated=gpu:NoSchedule
  -  Add toleration to ML pods: allow only them on GPU nodes

### How Do You Secure Inter-Pod Communication?

Best Practices:

NetworkPolicies:
- Define allowed traffic per namespace/app
- --> - from: - podSelector: matchLabels: app: frontend
     
mTLS (Mutual TLS):
- Use Istio or Linkerd for encrypted, authenticated traffic
  
TLS Secrets for APIs:
- Use Kubernetes Secrets + sidecar (e.g., cert-manager)

JWT/OAuth for internal APIs:
- Authenticate requests even inside the cluster
  
