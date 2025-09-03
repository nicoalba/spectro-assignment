# Debug Operations in Kubernetes

`kubectl` is the command-line interface (CLI) that is used to interact with Kubernetes (K8s). The `kubectl` CLI commmunicates with the K8s API server.

## Commands

Kubernetes contains several commands. We recommend when debugging you start with kubectl get pods, then `kubectl logs` and lastly we can use `kubectl exec` to explore the inside of the container and review other log files or configurations.

### kubectl get pods

This command is used to get a list of all pods that are available and what their status is. Just rememember that when you use this command that you may have to specify the `namespace`.

```shell
kubectl get pods --namespace
```

### kubectl logs

In Azure, kubernetess is available, just like other cloud providers. This command is used to retrive the logs of a specific pod - do use this when you have to review logs or need to debug a container.

### `kubectl exec`

A command that we can use to debug a container from the inside or to explore the the enviroment of the container itself.

### `kubectl debug`

Another option to considering when debugging a container. This command can be used to create a clone of a pod that does not terminate if an error is experienced inside the container.


# References

- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-
- [What is Kubernetes](https://kubernetes.io/docs/concepts/overview/)