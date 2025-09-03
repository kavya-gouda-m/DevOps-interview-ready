### The Big Picture — How Kubernetes Schedules a Pod
At a high level, Kubernetes Scheduler picks a node for your Pod in 3 main phases:

- Filtering → Who is eligible?
- Scoring → Who is better?
- Selecting → Who finally wins?

#### Filtering Nodes — Finding Candidates Fast

First phase is to filter the nodes in the cluster and find a feasible set of nodes for a pod to schedule.

The scheduler filters nodes based on:


- Node Affinity / Tolerations
- Resources (CPU, Memory)
- Custom Constraints

But there’s an important optimization most people don’t know: numFeasibleNodesToFind

Instead of scanning all nodes, Kubernetes might stop filtering after finding enough feasible nodes.

Why? → Performance in large clusters!
```
if foundEnoughFeasibleNodes {
    stop filtering
}
```

By default:

- It targets 50% of all nodes, capped to at least 100.
- Configurable via percentageOfNodesToScore


https://medium.com/@hmusicofficial27/inside-kubernetes-scheduler-what-really-happens-before-your-pod-lands-on-a-node-99e9aeb829a1
- 
