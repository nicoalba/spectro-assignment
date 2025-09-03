# Debug Operations in Kubernetes

`kubectl` is the command-line interface (CLI) tool used to interact with and manage Kubernetes (K8s) clusters. With `kubectl`, you can commmunicate with the K8s API server and execute commands against a K8s cluster to deploy applications, manage resources, or view logs.

`kubectl` is an essential tool for debugging

## Recommended Debugging Workflow

When debugging, we recommend this workflow:

1. `kubectl get pods`: Gives you an inventory of the pods in your K8s cluster.
2. `kubectl describe pod`: Gives you a snapshot of a specific pod's current state and configuration.
3. `kubectl logs`: Gives you the standard output (`stdout`) and standard error (`stderr`) streams from a container running a pod.
4. `kubectl exec`: Lets you execute a command directly inside a running container with a K8s pod.

## Commmands

This section expands on `kubectl` commands used for debugging, how to use them, and what flags are available.

### `kubectl get pods`

This command is used to get a list of all pods that are available and what their status is. Just rememember that when you use this command that you may have to specify the `namespace`.

```shell
kubectl get pods --namespace
```

### `kubectl logs`

In Azure, kubernetess is available, just like other cloud providers. This command is used to retrive the logs of a specific pod - do use this when you have to review logs or need to debug a container.

### `kubectl exec`

A command that we can use to debug a container from the inside or to explore the the enviroment of the container itself, including log files and configurations.

### `kubectl debug`

Another option to considering when debugging a container. This command can be used to create a clone of a pod that does not terminate if an error is experienced inside the container.

# References

- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-
- [What is Kubernetes](https://kubernetes.io/docs/concepts/overview/)