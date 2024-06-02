---
title: "How to Troubleshoot Kubernetes Clusters Effectively"
datePublished: Sun Jun 02 2024 08:01:23 GMT+0000 (Coordinated Universal Time)
cuid: clwx96hm4000h09jye25v1ybu
slug: how-to-troubleshoot-kubernetes-clusters-effectively
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1717315012011/f93350e6-033a-4a9f-b7a6-e35b992bc900.avif
tags: aws, kubernetes, developer, devops, devops-articles, hasg

---

Essential Tips and Techniques for Diagnosing and Resolving Kubernetes Issues with me!

## Viewing Basic Cluster Information

The first step in troubleshooting container issues is to get basic information about the Kubernetes worker nodes and services running on the cluster.

### List Worker Nodes

To see a list of worker nodes and their status, run the following command:

```sh
kubectl get nodes --show-labels
```

This will output something like:

```sh
NAME      STATUS    ROLES    AGE     VERSION        LABELS
worker0   Ready     <none>   1d      v1.13.0        ...,kubernetes.io/hostname=worker0
worker1   Ready     <none>   1d      v1.13.0        ...,kubernetes.io/hostname=worker1
worker2   Ready     <none>   1d      v1.13.0        ...,kubernetes.io/hostname=worker2
```

### Get Cluster Services Information

To get information about the services running on the cluster, use:

```sh
kubectl cluster-info
```

The output will look like this:

```sh
Kubernetes master is running at https://104.197.5.247
elasticsearch-logging is running at https://104.197.5.247/api/v1/namespaces/kube-system/services/elasticsearch-logging/proxy
kibana-logging is running at https://104.197.5.247/api/v1/namespaces/kube-system/services/kibana-logging/proxy
kube-dns is running at https://104.197.5.247/api/v1/namespaces/kube-system/services/kube-dns/proxy
```

## Retrieving Cluster Logs

For deeper diagnostics, you need access to logs on the nodes. Here’s where you can find them:

| Node Type | Component | Where to Find Logs |
| --- | --- | --- |
| Master | API Server | /var/log/kube-apiserver.log |
| Master | Scheduler | /var/log/kube-scheduler.log |
| Master | Controller Manager | /var/log/kube-controller-manager.log |
| Worker | Kubelet | /var/log/kubelet.log |
| Worker | Kube Proxy | /var/log/kube-proxy.log |

## Common Cluster Failure Scenarios and Resolutions

Here are several common issues that can occur in a Kubernetes cluster, their impacts, and how to resolve them.

### API Server VM Shuts Down or Crashes

* **Impact**: You can’t start, stop, or update pods and services.
    
* **Resolution**: Restart the API server VM.
    
* **Prevention**: Set the API server VM to restart automatically and set up high availability for the API server.
    

### Control Plane Service Shuts Down or Crashes

* **Impact**: Same as API Server VM shutdown since services like Replication Controller Manager and Scheduler are collocated with the API Server.
    
* **Resolution**: Restart the affected services.
    
* **Prevention**: Same as API Server VM Shuts Down.
    

### API Server Storage Lost

* **Impact**: API Server will fail to restart after shutting down.
    
* **Resolution**: Fix the storage issue, recover the API Server state from backup, and restart it.
    
* **Prevention**: Keep a snapshot of the API Server ready. Use reliable storage solutions like Amazon Elastic Block Storage (EBS).
    

### Worker Node Shuts Down

* **Impact**: Pods on the node stop running. The scheduler will try to run them on other nodes, reducing the cluster’s capacity.
    
* **Resolution**: Identify the issue, bring the node back up, and register it with the cluster.
    
* **Prevention**: Use a replication controller or a service to ensure users are not affected by node failures. Design applications to be fault-tolerant.
    

### Kubelet Malfunction

* **Impact**: If the kubelet crashes, you can’t start new pods on that node. Existing pods might be deleted, and the node will be marked unhealthy.
    
* **Resolution**: Same as Worker Node Shuts Down.
    
* **Prevention**: Same as Worker Node Shuts Down.
    

### Unplanned Network Partitioning

* **Impact**: Nodes in the affected partition are considered down by the master, and they can’t communicate with the API Server.
    
* **Resolution**: Reconfigure the network to restore communication between all nodes and the API Server.
    
* **Prevention**: Use a networking solution that can automatically reconfigure network parameters.
    

### Human Error by Cluster Operator

* **Impact**: Accidental commands or misconfigurations can cause loss of pods, services, or control plane components, disrupting service.
    
* **Resolution**: Restore the API Server state from backup.
    
* **Prevention**: Implement automatic review and correction of configuration errors in your Kubernetes clusters.
    

By understanding and utilizing these troubleshooting steps, you'll be better equipped to manage and resolve issues within your Kubernetes clusters. Regular monitoring, proactive maintenance, and having a solid recovery plan are key to ensuring the stability and reliability of your containerized applications. With this guide, you can confidently navigate and mitigate common failure scenarios, keeping your Kubernetes environment running smoothly. I hope you like this blog and Do share it with your friends.  
Thanks For Reading!