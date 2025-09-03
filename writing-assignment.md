# Debug Operations in Kubernetes

`kubectl` is the command-line interface (CLI) tool used to interact with and manage Kubernetes (K8s) clusters. With `kubectl`, you can commmunicate with the K8s API server and execute commands against a K8s cluster to deploy applications, manage resources, or view logs.

`kubectl` is also an essential tool for dianogosing and debugging issues with your applications. With `kubectl`, you can access and investigate pod status, configurations, and container logs. For advanced users, you can even attach temporary debugging containers to running workloads.

This guide outlines the most common `kubectl` commands and a recommended workflow for effective troubleshooting.

- [Debug Operations in Kubernetes](#debug-operations-in-kubernetes)
  - [Recommended Debugging Workflow](#recommended-debugging-workflow)
  - [`kubectl get pods`](#kubectl-get-pods)
    - [Example output and explanation](#example-output-and-explanation)
  - [`kubectl describe pod`](#kubectl-describe-pod)
    - [Example output and explanation](#example-output-and-explanation-1)
  - [`kubectl logs`](#kubectl-logs)
    - [Example output and explanation](#example-output-and-explanation-2)
  - [`kubectl exec`](#kubectl-exec)
    - [Example output and explanation](#example-output-and-explanation-3)
  - [`kubectl debug`](#kubectl-debug)
    - [Example output and explanation](#example-output-and-explanation-4)
  - [References](#references)

## Recommended Debugging Workflow

When debugging, we recommend this workflow:

1. `kubectl get pods`: Gives you an inventory of the pods in your K8s cluster.
2. `kubectl describe pod`: Gives you a snapshot of a specific pod's current state and configuration.
3. `kubectl logs`: Gives you the standard output (`stdout`) and standard error (`stderr`) streams from a container running a pod.
4. `kubectl exec`: Lets you execute a command directly inside a running container with a K8s pod.
5. `kubetl debug`: Creates a temporary debug container to a running pod for advanced troubleshooting when `exec` is insufficient.

## `kubectl get pods`

Use this command to get a list of all available pods in a given [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/). It lists their status, health, and age. This command is typically the first one executed for pod health checks.

The command executes against your current context's namespace unless you specify a namespace with `--namespace`:

```shell
kubectl get pods --namespace
```

### Example output and explanation

```shell
$ kubectl get pods
NAME                                       READY   STATUS             RESTARTS      AGE
frontend-deployment-78b7999885-2sk6j       1/1     Running            0             13m
mysql-76c74d896b-4g8b8                     1/1     Running            0             13m
webapp-deployment-559d86b864-sk7r9         0/1     CrashLoopBackOff   4             2m
```

- `NAME`: The name of the pod
- `READY`: The number of ready containers / the number of total containers
- `STATUS`: Current state of the pod: `Running`, `Pending`, `Succeeded`, `Failed`, or `CrashLoopBadOff`
- `RESTARTS`: How many times the containers in the pod have restarted
- `AGE`: The amount of time elapsed since creation of the pod

For more info, available flags, and example commands, see [`get` on K8s docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get).

## `kubectl describe pod`

Use this command to get a detailed view of a specific pod or a set of pods in a K8s cluster. While `kubectl get pods` offers a summary, `describe` gathers information from multiple API endpoints to present a comprehensive snapshot of a pod's current state and configuration. It is an essential tool for troubleshooting pod-related issues.

### Example output and explanation

```shell
$ kubectl describe pod webapp-deployment-559d86b864-sk7r9
Name:             webapp-deployment-559d86b864-sk7r9
Namespace:        default
Priority:         0
Node:             k8s-worker-2/10.128.0.12
Start Time:       Wed, 03 Sep 2025 13:20:45 +0000
Labels:           app=webapp
                  pod-template-hash=559d86b864
Status:           Running
IP:               10.42.0.14
Containers:
  webapp:
    Container ID:   containerd://a1b2c3d4e5f6g7h8i9j0
    Image:          my-webapp:v2
    Image ID:
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       ContainerCreating
      Exit Code:    1
    Ready:          False
    Restart Count:  5
    ...
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  12m                  default-scheduler  Successfully assigned default/webapp-deployment-559d86b864-sk7r9 to k8s-worker-2
  Normal   Pulling    5m (x3 over 11m)     kubelet            Pulling image "my-webapp:v2"
  Warning  Failed     5m (x3 over 11m)     kubelet            Failed to pull image "my-webapp:v2": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/my-webapp:v2": no matching manifest for linux/amd64 in the manifest list entries
  Warning  BackOff    3m (x5 over 10m)     kubelet            Back-off pulling image "my-webapp:v2"
```

For each container in the pod, this output lists:

- Metadata and status:
  - Name, Namespace, Labels
  - Node: The specific worker node
  - Status: The pod's current state
  - IP: The pod's internal IP address
- Containers:
  - Image: The container image being used
  - State: The current state of the container
  - Ready: If the container is ready to serve traffic
  - Restart count: How many times the container has been restarted
  - Resource requests/limits: CPU and memory reources requested and limited for the container
- Volumes:
  - ConfigMaps, Secrets, Persistent Volumes
- Events:
  - Normal events
  - Warning events

For more info, available flags, and example commands, see [`describe` on K8s docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe).


## `kubectl logs`

Use this command to retrieve and display the standard output (`stdout`) and standard error (`stderr`) streams from a container running in a pod. While `kubectl get pods` tells you if a pod is healthy, `kubectl logs` gives you visibility into what the application inside the container is doing.

### Example output and explanation

```shell
$ kubectl logs frontend-deployment-78b7999885-2sk6j
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: /etc/nginx/conf.d/default.conf is not a file or does not exist
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
10.42.0.1 - - [03/Sep/2025:13:20:45 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36" "-"
10.42.0.1 - - [03/Sep/2025:13:20:50 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36" "-"
```

The logs show the raw output of what your application is doing inside the container. You'll typically see:

- Startup and system messages: Initialization messages, system and informational output, warnings and errors
- Runtime logs: Application-specific data, error stack traces and exceptions, debugging output

For more info, available flags, and example commands, see [`logs` on K8s docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs).

## `kubectl exec`

Use the `kubectl exec` command to execute a command directly inside a running container within a pod. It provides interactive access to the container's shell and environment, which is powerful for live debugging and troubleshooting.

### Example output and explanation

```shell
$ kubectl exec frontend-deployment-78b7999885-2sk6j -- cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
10.42.0.12 frontend-deployment-78b7999885-2sk6j
```

The output depends on the command you execute once you enter the interactive session.

For more info, available flags, and example commands, see [`exec` on K8s docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#exec).

## `kubectl debug`

Use this command for advanced troubleshooting, when all other methods have proven insufficient. These are the most common use cases:

- **Create an ephemeral (temporary) container in a running pod**: `kubectl debug` creates and adds a temporary, interactive container to an existing pod that lets you inspect the running processes and files of the original container without modifying or restarting it.
- **Copy and modify a pod**: If ephemeral containers are not enabled in your cluster or you need to test a specific change, `kubectl debug` can create a copy of the pod. You can then add a debugging container to the pod copy for testing. It's an isolated way to test a fix without affecting the live workload.
- **Debug a cluster node**: `kubectl debug` can create a new pod that runs directly on a specific node, mounting the host's filesystem at `/host`. This creates an SSH-like shell on the node itself, letting you troubleshoot node-level issues like networking problems, log analysis, or filesystem issues without a true SSH connection.

### Example output and explanation

The output of `kubectl debug` depends on the debugging mode you use. The most common use case is adding an interactive ephemeral container to a running pod:

```
$ kubectl debug -it mypod --image=busybox --target=main-app
Defaulting debug container name to debugger-d4fsh.
If you don't see a command prompt, try pressing enter.
/ # ls -l /proc/1/
total 0
dr-xr-xr-x 8 root root 0 Sep  3 15:00 attr
-r-------- 1 root root 0 Sep  3 15:00 autogroup
-r-------- 1 root root 0 Sep  3 15:00 cgroup
# ... (rest of the /proc/1 directory output) ...
/ # exit
exit
```

For more information, see [kubectl debug](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#debug).

## References

- [Kubectl commands (Official Docs)](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
- [Kubernetes Overview (Official Docs)](https://kubernetes.io/docs/concepts/overview/)
