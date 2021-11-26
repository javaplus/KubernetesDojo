# ~~ Running in Kubernetes ~~

Hopefully you understand a little about containers and images because we will be learning how to run your containers in Kubernetes.  Kubernetes can orchestrate and schedule your container workloads in very flexible ways, but at the end of the day it's going to invoke a Docker or other [CRI compatible runtime](https://www.opencontainers.org/) to run your application.

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

With the kubectl command, you will be creating and interacting with several Kubernetes **Resources**.  The first resource you've now seen is the [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/).  We wont' go too deep on all the specifics, but a Pod is where the configuration for your running container resides.  It turns out you can run multiple containers in a single Pod, but that is outside the scope of this exercise.  To learn more about how and why you would want to do this, search around for Sidecar and Ambassador patterns.

Let's see what happens when we delete a Pod (which is one way to brute force simulate a failed container).  Run the following command, and then keep running the **kubectl get pods** command to see what happens to your pod.  Things will happen fast!

```bash
kubectl delete pod mynginx
```
This should delete your Pod which can be seen by running the **kubectl get pods** command again.


## Connecting to your Running Pod.


