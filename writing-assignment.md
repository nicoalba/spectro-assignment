# Debug Operations in Kubernetes

`kubectl` is the command-line interface (CLI) tool used to interact with and manage Kubernetes (K8s) clusters. With `kubectl`, you can commmunicate with the K8s API server and execute commands against a K8s cluster to deploy applications, manage resources, or view logs.

`kubectl` is an essential tool for debugging

## Recommended Debugging Workflow

When debugging, we recommend this workflow:

1. `kubectl get pods`: Gives you an inventory of the pods in your K8s cluster.
2. `kubectl describe pod`: Gives you a snapshot of a specific pod's current state and configuration.
3. `kubectl logs`: Gives you the standard output (`stdout`) and standard error (`stderr`) streams from a container running a pod.
4. `kubectl exec`: Lets you execute a command directly inside a running container with a K8s pod.
5. `kubetl debug`: Creates a temporary debug container to a running pod for advanced troubleshooting when `exec` is insufficient.

This section expands on `kubectl` commands used for debugging, how to use them, and what flags are available.

## `kubectl get pods`

Use this command to get a list of all available pods in a given namespace. It lists their status, health, and age. This command is typically the first one executed for pod health checks.

The command will execute against your current context's namespace unless you specify a namespace with `--namespace`:

```shell
kubectl get pods --namespace
```

Example output:

```shell
$ kubectl get pods
NAME                                       READY   STATUS             RESTARTS      AGE
frontend-deployment-78b7999885-2sk6j       1/1     Running            0             13m
mysql-76c74d896b-4g8b8                     1/1     Running            0             13m
webapp-deployment-559d86b864-sk7r9         0/1     CrashLoopBackOff   4             2m
```

Output explanation:

- `NAME`: The name of the pod
- `READY`: The number of ready containers / the number of total containers.
- `STATUS`: Current state of the pod: `Running`, `Pending`, `Succeeded`, `Failed`, or `CrashLoopBadOff`.
- `RESTARTS`: How many times the containers in the pod have restarted.
- `AGE`: The amount of time elapsed since creation of the pod.

### Flags for `kubectl get pods`

| Flag               | Short form | Description                                                                                   | Example                                                  |
| ------------------ | ---------- | --------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| `--namespace`      | `-n`       | Specifies a particular namespace to search for pods.                                          | `kubectl get pods -n kube-system`                        |
| `--all-namespaces` | `-A`       | Lists pods across all namespaces in the cluster.                                              | `kubectl get pods -A`                                    |
| `--output=wide`    | `-o wide`  | Adds extra columns to the output, such as the pod's IP address and the node it is running on. | `kubectl get pods -o wide`                               |
| `--selector`       | `-l`       | Filters pods based on a specified label.                                                      | `kubectl get pods -l app=nginx`                          |
| `--watch`          | `-w`       | Watches for changes and continuously streams updates to the terminal.                         | `kubectl get pods -w`                                    |
| `--field-selector` |            | Filters pods based on one or more field selectors.                                            | `kubectl get pods --field-selector=status.phase=Failed`  |
| `--sort-by`        |            | Sorts the list of pods based on a specified JSONPath expression.                              | `kubectl get pods --sort-by=.metadata.creationTimestamp` |
| `--show-labels`    |            | Adds an extra column that shows the labels for each pod.                                      | `kubectl get pods --show-labels`                         |
| `--output`         | `-o`       | Specifies an output format other than the default table (e.g., `yaml`, `json`).               | `kubectl get pods -o yaml`                               |


## `kubectl describe pod`

The `kubectl describe pod` command provides a detailed, human-readable view of a specific pod. While `kubectl get pods` offers a summary, `describe` gathers information from multiple API endpoints to present a comprehensive snapshot of a pod's current state and configuration. It is an essential tool for troubleshooting pod-related issues.

Example output:

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

Ouput explanation:

For each container in the pod, this output lists important information:

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

### Flags for `kubectl describe pod`

| Flag               | Short form | Description                                            | Example                                       |
| ------------------ | ---------- | ------------------------------------------------------ | --------------------------------------------- |
| `--namespace`      | `-n`       | Specifies the namespace of the pod(s) to describe.     | `kubectl describe pod my-pod -n my-namespace` |
| `--filename`       | `-f`       | Describes resource(s) from a local manifest file.      | `kubectl describe -f my-pod.yaml`             |
| `--selector`       | `-l`       | Describes all pods matching a specific label selector. | `kubectl describe pods -l app=nginx`          |
| `--all-namespaces` | `-A`       | Describes pods across all namespaces in the cluster.   | `kubectl describe pods -A`                    |


## `kubectl logs`

This command is used to retrieve and display the standard output (`stdout`) and standard error (`stderr`) streams from a container running in a pod. While `kubectl get pods` tells you if a pod is healthy, `kubectl logs` gives you visibility into what the application inside the container is doing.

Example output:

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

The logs the raw output of what your application is doing inside the container. You'll typically see:

- Startup and system messages: Initialization messages, system and informational output, warnings and errors
- Runtime logs: Application-specific data, error stack traces and exceptions, debugging output

Output explanation:

### Flags for `kubectl logs`

| Flag               | Short form | Description                                                                               | Example                                |
| ------------------ | ---------- | ----------------------------------------------------------------------------------------- | -------------------------------------- |
| `--container`      | `-c`       | Specifies a container name in a multi-container pod. Required if a pod has more than one. | `kubectl logs my-pod -c my-container`  |
| `--follow`         | `-f`       | Streams new log entries to the terminal in real time (like `tail -f`).                    | `kubectl logs -f my-pod`               |
| `--previous`       | `-p`       | Prints logs for the **previous** instance of the container (useful after restarts).       | `kubectl logs my-pod --previous`       |
| `--tail`           |            | Displays only the most recent number of log lines.                                        | `kubectl logs my-pod --tail=50`        |
| `--since`          |            | Shows logs from a relative duration (e.g., `5s`, `2m`, `3h`).                             | `kubectl logs my-pod --since=10m`      |
| `--timestamps`     |            | Adds a timestamp to each log entry (helps correlate events).                              | `kubectl logs my-pod --timestamps`     |
| `--all-containers` |            | Prints logs for **all containers** in the pod.                                            | `kubectl logs my-pod --all-containers` |
| `--selector`       | `-l`       | Retrieves logs from all pods matching the label selector.                                 | `kubectl logs -l app=nginx`            |


## `kubectl exec`

Use the `kubectl exec` command allows you to execute a command directly inside a running container within a pod. It provides interactive access to the container's shell and environment, which is powerful for live debugging and troubleshooting

Example output:

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

Output explanation:

The output depends on the command you execute once you enter the interactive session.

### Flags for `kubectl exec`

| Flag          | Short form | Description                                                           | Example                                            |
| ------------- | ---------- | --------------------------------------------------------------------- | -------------------------------------------------- |
| `--container` | `-c`       | Specifies a container name in a multi-container pod (required if >1). | `kubectl exec my-pod -c my-container -- /bin/bash` |
| `--stdin`     | `-i`       | Keeps standard input open for the container (interactive sessions).   | `kubectl exec -i my-pod -- /bin/bash`              |
| `--tty`       | `-t`       | Allocates a pseudo-terminal (pair with `-i` for interactive shells).  | `kubectl exec -t my-pod -- /bin/bash`              |
| `--quiet`     | `-q`       | Suppresses kubectl’s own output; shows only the command’s output.     | `kubectl exec -q my-pod -- date`                   |
| `--namespace` | `-n`       | Specifies the namespace of the target pod.                            | `kubectl exec -n my-namespace my-pod -- ls`        |

## `kubectl debug`

kubectl debug offers multiple powerful debugging modes by creating new containers with enhanced capabilities:

- Create an ephemeral container in a running pod: This is the most common use case. kubectl debug creates and adds a temporary, interactive container to an existing pod. This debug container can be based on a debug-optimized image (like busybox or nicolaka/netshoot) and can share the process namespace with the target container. This allows you to inspect the running processes and files of the original container without modifying or restarting it.
- Copy and modify a pod: If ephemeral containers are not enabled in your cluster or you need to test a specific change, kubectl debug can create a copy of the pod. This new pod can have its image or other attributes changed for testing, and a debug container is added. It's an isolated way to test a fix without affecting the live workload.
- Debug a cluster node: kubectl debug can create a new pod that runs directly on a specific node, mounting the host's filesystem at /host. This effectively provides a "live SSH-like" shell on the node itself, allowing you to troubleshoot node-level issues like networking problems, log analysis, or filesystem issues.

The output of kubectl debug is not fixed, as it depends on the debugging mode you use. The most common use case is adding an interactive ephemeral container to a running pod

Example output:

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

### Flags for `kubectl debug`

| Flag                | Short form | Description                                                                                   | Example                                                       |
| ------------------- | ---------- | --------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| `--image`           |            | Specifies the container image to use for the debug container.                                 | `kubectl debug mypod --image=busybox`                         |
| `--target`          |            | Targets an existing container; the debug container shares that container’s process namespace. | `kubectl debug mypod --image=busybox --target=main-app`       |
| `--share-processes` |            | When used with `--copy-to`, enables process namespace sharing in the copied pod.              | `kubectl debug mypod --copy-to=my-debug --share-processes`    |
| `--copy-to`         |            | Creates a copy of the specified pod with changes applied for debugging.                       | `kubectl debug mypod --copy-to=my-debug --image=debian`       |
| `--container`       | `-c`       | Names the **debug** container to be created.                                                  | `kubectl debug mypod --container=my-debugger`                 |
| `--tty`             | `-t`       | Allocates a pseudo-terminal (combine with `-i` for interactive shells).                       | `kubectl debug mypod -it`                                     |
| `--stdin`           | `-i`       | Keeps standard input open (interactive mode).                                                 | `kubectl debug mypod -it`                                     |
| `--profile`         |            | Applies a security profile for node debugging (e.g., `sysadmin` for elevated privileges).     | `kubectl debug node/mynode --image=ubuntu --profile=sysadmin` |
| `--replace`         |            | With `--copy-to`, deletes the original pod after creating/debugging the copy.                 | `kubectl debug mypod --copy-to=my-debug --replace`            |
| `node/<node-name>`  |            | Targets a **node** directly, creating a privileged debug pod scheduled to that node.          | `kubectl debug node/mynode --image=ubuntu`                    |

## References

- [Kubectl commands (Official Docs)](https://kubernetes.io/docs/reference/generated/kubectl/)kubectl-commands#-strong-getting-started-strong-
- [Kubernetes Overview (Official Docs)](https://kubernetes.io/docs/concepts/overview/)
