## Kubernetes Cluster Maintenance
Cluster maintenance ensures a Kubernetes environment remains healthy, secure, and performant. It involves monitoring, upgrades, resource management, and configuration changes.

#### 1. Node Maintenance

| Task               | Command/Method                                                         | Description                               |
| ------------------ | ---------------------------------------------------------------------- | ----------------------------------------- |
| Drain node         | `kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data` | Safely evict workloads before maintenance |
| Mark unschedulable | `kubectl cordon <node-name>`                                           | Prevent new pods from being scheduled     |
| Mark schedulable   | `kubectl uncordon <node-name>`                                         | Allow pods to be scheduled again          |
| Delete node        | `kubectl delete node <node-name>`                                      | Remove a node from cluster registry       |

#### Draining a Node
- For situations where the recovery time is uncertain, the safer method is to drain the node. Draining involves gracefully terminating the pods running on that node so they are recreated on other nodes.
- At the same time, draining marks the node as unschedulable (cordoned), preventing new pods from being scheduled on it until explicitly allowed.
- After the node's workloads have been safely relocated, you can reboot the node. When it comes back online, it remains unschedulable until you uncordon it, allowing new pods to be scheduled

#### Cordon vs. Drain
In addition to draining and uncordoning, Kubernetes provides the cordon command. Unlike drain, cordon only marks a node as unschedulable without terminating or relocating the currently running pods. This ensures that no new pods will be scheduled on the node.

        Be cautious when using cordon on a node with critical workloads. Since existing pods remain and new pods cannot be scheduled, your application might become overloaded or experience unexpected behavior.