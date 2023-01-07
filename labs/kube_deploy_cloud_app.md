# ~~ Working with Deployments ~~

Now that you've worked a little with the Pod resource to run a container in Kubernetes, we will look at another common resource in Kubernetes the [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).  A Pod is great for running your container, but the Deployment resource is one of the reasons Kubernetes can be really valuable for production deployments.  Your containers will always run within a Pod, but a deployment manages running pods.  Let's explore Deployments with more hands-on exercises.

## Create a Deployment in Kubernetes

In a console window run the following to run the `javaplus/cloud-native-demo` image as a container in Kubernetes:

```bash
kubectl create deployment cn-demo --image=javaplus/cloud-native-demo
```

Now you can use a **kubectl get** command to see what was created. In this case, we want to see the running container which in Kubernetes is always wrapped in a Pod.  So, issue the following command to see all the current pods.

```
kubectl get pods
```

You should see a Pod that starts with the name `cn-demo-` show up.  Behind the scenes, Kubernetes is retrieving the image and executing it using Docker (in the case of the Kubernetes instance provided by Docker Desktop).  If all goes well, you should eventually see that the Pod is up and running with output in the dashboard similar to the following:

```bash
NAME                           READY   STATUS    RESTARTS   AGE
pod/cn-demo-759dc65498-j2mm6   1/1     Running   0          22s
```

## Play with the new Deployment

Let's see what happens when we delete a Pod (which is one way to brute force simulate a failed container).  Run the following command, filling in the `cn-demo-***` with the unique name Kubernetes assigned your Pod, and then keep running the **kubectl get pods** command to see what happens to your pod.  Things will happen fast!

```bash
kubectl delete pod cn-demo-***
```


You deleted **that** specific Pod, but then another one with a new name showed up in its place.  That Pod is running the exact same image, and the Deployment in the Kubernetes cluster is making sure at least 1 of your Pods is running.  That [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) you created is always working to ensure the number of Pods running for a given `Deployment` match its configured `replica` count.

In fact, you can see it by running **kubectl get deploy**.  

This should show the `cn-demo` deployment happily reporting that `1/1` or "1 of 1" Pods are currently available.
```
D:\workspaces\DockerKubesDojo>kubectl get deploy
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
cn-demo   1/1     1            1           9m10s

```

If you're thinking "a Deployment must be involved in how Horizontal Scaling works", you'd be right.  Let's attempt that next.

```bash
kubectl scale deployment cn-demo --replicas 3
```

Run your **kubectl get pods** again.  Do you see 3 Pods for `cn-demo-***` running?  Your `Deployment` should report `3/3` after some time as well if you run the **kubectl get deploy**

Try some other scenarios to see how Kubernetes behaves:
* Delete one of the newly created `Pods` with `kubectl delete ...`
* Scale the `Deployment` in and out, using different numbers for `--replicas`
* Do you notice anything interesting with which Pods Kubernetes decides to keep when you scale in?  Look at the time the Pod has been running.
* If you need to delete the **deployment** altogether (which will remove the pods as well) you can use this command:
```
kubectl delete deploy cn-demo

```
NOTE: If you delete it, you will have to restart it for the next labs.
Remember the command to start it is:
```bash
kubectl create deployment cn-demo --image=javaplus/cloud-native-demo:1
```

### Stretch Goal

Get up and stretch!!!  Just kidding... ok maybe that's not a bad idea... but to play more with kubernetes, let's see if we can learn how to connect to one of the running containers and get a shell so we can poke around and see the files that are in our running container.  What we will do is use the **kube exec** command to get a bash shell into one of our pods.

So, make sure you have at least one pod running, and then use the kube exec command to get a shell into the container.
The format of the kube exec command is like this:
```bash
kubectl exec -it <pod name> -- /bin/bash
```

NOTE: Look at the [offical documentation here](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#exec) to see what the **'-it'** is doing.  

Once you get a bash prompt issue a **pwd** command to see what the current working directory is.
Do you know why this is the working directory?

Look at the Dockerfile here [Dockerfile](https://github.com/javaplus/KubernetesDojo/blob/master/Dockerfile) used to create this image and see if you can see why.

Now, try to actually create a new file in that directory.  You can just do a simple echo command like this:
```bash
echo "hello" > hello.txt
```
After creating the file, exit out of the shell session by simply typing **exit**.

Now, reconnect to the pod again and make sure your file is still there. (It should be).

Now, let's delete the pod and let the deployment spin up a new pod.

When your new pod finishes starting, exec into it and see if your file is still there.
Can you figure out why or why not?

<details>
  <summary>Click to expand for Answers</summary>
 
 #### Explanation
  
 - Why is the working directory "/usr/src/app"?
  - Because the [Dockerfile on line 3](https://github.com/javaplus/KubernetesDojo/blob/a8003fc12cc98fee5bb87a2840d377b75d7239f7/Dockerfile) set the "WORKDIR" to "/usr/src/app" 
  - What does the **'-it'** do with the exec command?
    - The 'i' says pass the STDIN of your command prompt to the container
    - The 't' says the STDIN is a TTY
    - Most think of the **'-it'** just as an interactive terminal because that's what it produces.
    
  - Why did the hello.txt file disappear after deleting the pod and letting the deployment create a new one?
    - Because the container in the Pod is an instance of the image you specify.  When we added the file, we added it to that specific running instance... think of it like modifying temporary memory or modifying an instance of a class.  Once we delete that Pod that deletes that instance of the container.  Then the deployment starts up a new Pod which creates a new instance of the container off of the image we specified.  The only thing that's going to be in the running container is what we specified in the image definition (assuming we don't give special commands to the start up). 
  
  
</details>


