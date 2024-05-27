---
title: "Troubleshooting Kubernetes Pods: A Complete Guide"
datePublished: Mon May 27 2024 18:30:26 GMT+0000 (Coordinated Universal Time)
cuid: clwpb0c2j00050alfce55ee0r
slug: troubleshooting-kubernetes-pods-a-complete-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1716827178168/f9ddef83-23fa-4aa1-b537-96e98c2c683b.avif
tags: kubernetes, devops, k8s, troubleshooting, devops-articles, kubernetes-container

---

When facing issues with a Kubernetes pod that aren't quickly resolved, you'll need to dig deeper. Start by running:

```sh
kubectl describe pod [name]
```

### Understanding the `kubectl describe pod` Output

Here's an example output from the `describe pod` command:

```plaintext
Name:		nginx-deployment-1006230814-6winp
Namespace:	default
Node:		kubernetes-node-wul5/10.240.0.9
Start Time:	Thu, 24 Mar 2016 01:39:49 +0000
Labels:		app=nginx,pod-template-hash=1006230814
Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"default","name":"nginx-deployment-1956810328","uid":"14e607e7-8ba1-11e7-b5cb-fa16" ...
Status:		Running
IP:		10.244.0.6
Controllers:	ReplicaSet/nginx-deployment-1006230814
Containers:
  nginx:
    Container ID:	docker://90315cc9f513c724e9957a4788d3e625a078de84750f244a40f97ae355eb1149
    Image:		nginx
    Image ID:		docker://6f62f48c4e55d700cf3eb1b5e33fa051802986b77b874cc351cce539e5163707
    Port:		80/TCP
    QoS Tier:
      cpu:	Guaranteed
      memory:	Guaranteed
    Limits:
      cpu:	500m
      memory:	128Mi
    Requests:
      memory:		128Mi
      cpu:		500m
    State:		Running
      Started:		Thu, 24 Mar 2016 01:39:51 +0000
    Ready:		True
    Restart Count:	0
    Environment:        [none]
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-5kdvl (ro)
Conditions:
  Type          Status
  Initialized   True
  Ready         True
  PodScheduled  True
Volumes:
  default-token-4bcbi:
    Type:	Secret (a volume populated by a Secret)
    SecretName:	default-token-4bcbi
    Optional:   false
QoS Class:      Guaranteed
Node-Selectors: [none]
Tolerations:    [none]
Events:
  FirstSeen	LastSeen	Count	From					SubobjectPath		Type		Reason		Message
  ---------	--------	-----	----					-------------		--------	------		-------
  54s		54s		1	{default-scheduler }						Normal		Scheduled	Successfully assigned nginx-deployment-1006230814-6winp to kubernetes-node-wul5
  54s		54s		1	{kubelet kubernetes-node-wul5}	spec.containers{nginx}	Normal		Pulling		pulling image "nginx"
  53s		53s		1	{kubelet kubernetes-node-wul5}	spec.containers{nginx}	Normal		Pulled		Successfully pulled image "nginx"
  53s		53s		1	{kubelet kubernetes-node-wul5}	spec.containers{nginx}	Normal		Created		Created container with docker id 90315cc9f513
  53s		53s		1	{kubelet kubernetes-node-wul5}	spec.containers{nginx}	Normal		Started		Started container with docker id 90315cc9f513
```

#### Key Sections in `describe pod` Output

1. **Name**: Basic pod information, including the node it's running on, labels, and current status.
    
2. **Status**: The current state of the pod, which can be:
    
    * Pending
        
    * Running
        
    * Succeeded
        
    * Failed
        
    * Unknown
        
3. **Containers**: Details about containers within the pod (in this example, one container named nginx).
    
4. **Containers:State**: The status of each container, which can be:
    
    * Waiting
        
    * Running
        
    * Terminated
        
5. **Volumes**: Storage volumes, secrets, or ConfigMaps mounted by the pod's containers.
    
6. **Events**: Recent events, such as image pulls, container creations, and container starts.
    

### Continue Debugging Based on Pod State

#### Pod Stays Pending

If a pod remains in a Pending state, it may be unable to schedule onto a node. Check the Events section in the `describe pod` output for messages indicating scheduling issues, such as:

* **Insufficient resources**: The cluster may lack the necessary CPU or memory. You might need to delete some pods, add resources to nodes, or add more nodes.
    
* **Resource requirements**: The pod may have specific resource requirements. Try reducing these requirements to make the pod schedulable on more nodes.
    

#### Pod Stays Waiting

If a pod is in a Waiting state, it is scheduled but unable to run. Check the Events section for reasons, often related to image fetching errors. Check for:

* **Image name**: Ensure the image name in the pod manifest is correct.
    
* **Image availability**: Confirm the image is available in the repository.
    
* **Manual test**: Run `docker pull` on your local machine to verify image retrieval.
    

#### Pod Is Running but Misbehaving

If a pod is running but not behaving as expected, consider two common causes: errors in the pod manifest or a mismatch between local and API server manifests.

##### Checking for Errors in Your Pod Description

Errors in the pod manifest, such as incorrect nesting or typos, are common. Delete the pod and recreate it with:

```sh
kubectl apply --validate -f mypod1.yaml
```

This command will highlight syntax errors, such as:

```plaintext
46757 schema.go:126] unknown field: continers
46757 schema.go:129] this may be a false alarm, see https://github.com/kubernetes/kubernetes/issues/5786
pods/mypod1
```

##### Checking for Mismatch Between Local and API Server Manifests

If the API server's pod manifest differs from your local copy, it can cause unexpected behavior. Retrieve the API server's pod manifest with:

```sh
kubectl get pods/[pod-name] -o yaml > apiserver-[pod-name].yaml
```

Compare the local YAML with the API server's version:

1. **Local YAML has extra lines**: Indicates a mismatch. Delete the pod and rerun it with the local manifest.
    
2. **API Server YAML has extra lines**: Normal, as the API server may add lines over time. The issue likely lies elsewhere.
    
3. **Identical YAML files**: The issue lies elsewhere.
    

### Diagnosing Other Pod Issues

If the above methods don't resolve the issue, try these additional debugging methods:

#### Examining Pod Logs

Retrieve logs for a malfunctioning container with:

```sh
kubectl logs [pod-name] [container-name]
```

If the container has crashed, use the `--previous` flag to get its crash log:

```sh
kubectl logs --previous [pod-name] [container-name]
```

#### Debugging with Container Exec

Many container images include debugging utilities. Run commands inside a container with:

```sh
kubectl exec [pod-name] -c [container-name] -- [your-shell-commands]
```

#### Debugging with an Ephemeral Container

If the container has crashed or lacks debugging utilities, use an ephemeral container (supported in Kubernetes v1.18+):

```sh
kubectl debug -it [pod-name] --image=[image-name] --target=[pod-name]
```

Use the `--target` flag to enable communication with other containers in the pod. Note the ephemeral container name for further commands.

#### Running a Debug Pod on the Node

For deeper debugging, create a special pod on the node with host privileges (not recommended for production):

```sh
kubectl debug node/[node-name] -it --image=[image-name]
```

This pod runs a container with host IPC, Network, and PID namespaces, with the root filesystem mounted at `/host`. Delete the pod when done with:

```sh
kubectl delete pod [debug-pod-name]
```

Troubleshooting Kubernetes pods can be a complex task, but with the right approach and tools, you can efficiently diagnose and resolve issues. By understanding the `kubectl describe pod` output, checking pod states, examining logs, and using debugging tools like container exec and ephemeral containers, you can pinpoint the root cause of problems and ensure smooth operation. Keep this guide handy as a reference to streamline your debugging process and maintain a healthy Kubernetes environment. Happy debugging!🔨🤖🔧

Thanks for reading !