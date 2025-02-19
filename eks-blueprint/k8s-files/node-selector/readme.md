
# LABELING NODES

1. To label nodes run 

```bash
kubectl label nodes node_name key=value
```
2. then apply kubernetes files with node affinity or node selector

```bash
kubectl apply -f <file name>
```

3. To describe nodes and check for labels run either


```bash
kubectl get nodes --show-labels
```
or 

```bash
kubectl describe nodes <node name>
```


# DRAINING NODES

Node draining in Kubernetes is the process of gracefully evicting all the pods from a node, allowing for maintenance tasks or decommissioning without disrupting running applications. The `kubectl drain` command is commonly used for this purpose. Here's an overview of how node draining works in Kubernetes:

1. **Cordoning the Node:**
   - Before draining a node, Kubernetes "cords off" the node to prevent new pods from being scheduled onto it. This is done using the `kubectl cordon` command, which marks the node as unschedulable. As a result, the Kubernetes scheduler won't place any new pods on the cordoned node.

   ```bash
   kubectl cordon <node-name>
   ```

2. **Evicting Pods:**
   - After cordoning the node, the `kubectl drain` command is used to evict existing pods from the node. The drain process ensures that each pod is gracefully terminated, allowing it to complete any ongoing tasks and properly shut down.

   ```bash
   kubectl drain <node-name> --ignore-daemonsets
   ```

   - During the eviction process, Kubernetes communicates with the Kubernetes API server to trigger the rescheduling of the pods on other nodes in the cluster.

3. **Pod Preemption:**
   - If there are not enough resources available on other nodes to accommodate the evicted pods, Kubernetes may perform pod preemption. This involves preempting (evicting) lower-priority pods from other nodes to make room for the higher-priority pods being evicted from the drained node.

4. **Node Cleanup:**
   - After all the pods have been successfully evicted or rescheduled, the drained node is typically taken out of the cluster. This can involve decommissioning the node or performing maintenance tasks on it.

   ```bash
   kubectl uncordon <node-name>
   ```

   - The `kubectl uncordon` command marks the node as schedulable again, allowing new pods to be scheduled onto it.

Node draining is a critical operation for cluster maintenance, such as applying updates, replacing hardware, or addressing other issues. It ensures that the impact on running applications is minimized by gracefully handling pod evictions and rescheduling. It's important to note that node draining is a manual operation, and administrators typically initiate it when needed. Additionally, tools like Kubernetes Controllers for node maintenance and upgrades (e.g., `kured` for automatic node restarts during updates) can help automate some aspects of this process.


# Taints and Tolerations

Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes. One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints.

## You add a taint to a node using kubectl taint. For example,
```bash
kubectl taint nodes node1 key1=value1:NoSchedule
```

### places a taint on node node1. The taint has key key1, value value1, and taint effect NoSchedule. This means that no pod will be able to schedule onto node1 unless it has a matching toleration

## To remove the taint added by the command above, you can run:
```bash
kubectl taint nodes node1 key1=value1:NoSchedule-
```

## You specify a toleration for a pod in the PodSpec. Both of the following tolerations "match" the taint created by the kubectl taint line above, and thus a pod with either toleration would be able to schedule onto node1:

```bash
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
```

```bash
tolerations:
- key: "key1"
  operator: "Exists"
  effect: "NoSchedule"
```


## Here's an example of a pod that uses tolerations:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "example-key"
    operator: "Exists"
    effect: "NoSchedule"
```