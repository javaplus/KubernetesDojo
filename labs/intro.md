# ~~ Running in Kubernetes ~~

Hopefully you understand a little about containers and images because we will be learning how to run your containers in Kubernetes.  Kubernetes can orchestrate and schedule your container workloads in very flexible ways, but at the end of the day it's going to invoke a Docker or other [OCI compatible runtime](https://www.opencontainers.org/) to run your application.

## Run Your First Containers in Kubernetes

In a console window, run the following to run the `nginx` image as a container in Kubernetes:

```bash
kubectl run mynginx --image=nginx
```

Now you can use a **kubectl get** command to see what was created. In this case, we want to see the running container which in Kubernetes is always wrapped in a [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/).  So, issue the following command to see all the current pods.

```
kubectl get pods
```

You should see a Pod with the name `mynginx` show up.  Behind the scenes, Kubernetes is retrieving the image and executing it using Docker (in the case of the Kubernetes instance provided by Docker Desktop).  If all goes well, you should eventually see that the Pod is up and running with output in the dashboard similar to the following:

```bash
NAME                           READY   STATUS    RESTARTS   AGE
pod/mynginx   1/1     Running   0          22s
```

## 

With the kubectl command, you will be creating and interacting with several Kubernetes **Resources**.  The first resource you've now seen is the [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/).  We won't go too deep on all the specifics, but a Pod is where the configuration for your running container resides.  It turns out you can run multiple containers in a single Pod, but that is outside the scope of this exercise.  To learn more about how and why you would want to do this, search around for [Sidecar and Ambassador](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/) patterns.

Let's see how to delete your Pod now.  Run the following command, and then keep running the **kubectl get pods** command to see what happens to your pod.  Things will happen fast!

```bash
kubectl delete pod mynginx
```
This should delete your Pod which can be seen by running the **kubectl get pods** command again.


## Connecting to your Running Pod.

Let's see how to connect into your container running in your Kubernetes pod. Unlike, Docker where you can just use a "-p" option to publish a port to connect into it, in Kubernetes Pods are typically inaccessiable outside of the Kubernetes cluster without other Resources being created to route traffic.  It's a bit too early to be getting into the details about these other resources right now, so instead we will show you a great way to quickly be able to forward traffic from your local machine to the Pod.
This technique is great to test out a pod or troubleshoot connectivity issues.  To do this, we the [port-forward command](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward) to forward traffic from our local machine into the kubernetes cluster.  In this case the kubernetes cluster happens to be local as well, but on it's own virtual network.  Before we can forward traffic, we need a pod.  So, let's start up our nginx pod again:
```bash
kubectl run mynginx --image=nginx
```
Run **"kubectl get pods"** to make sure it's started.  
Once it's started, let's run the port forwarder to route traffic from your local machine into the kubernetes cluster.

```bash
kubectl port-forward pod/mynginx 8080:80
```
Now open up a browswer and go to http://localhost:8080 and you should see the NGINX welcome page.

Notice that the port-forward stays running and ferries requests from the port 8080 to the container port 80.
To break out of the port forward go ahead and hit "CTRL + C" to break out and stop the port forwarder.

[Port forwarding](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) is a great way to debug your resources in Kubernertes.

Go ahead and delete your pod before we move onto the next exercise.


