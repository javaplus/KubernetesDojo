# ~~ Infrastructure as Code ~~

## Clone this repo:
For many of the exercises beyond this point, you'll need to work with files in this repository.  So go ahead and clone this repo your machine now:
```bash
git clone https://github.com/javaplus/KubernetesDojo.git
```

> Note: Up to now, it didn't matter which directory you were running kubectl commands from, since we were just getting details on our Kubernetes environment, or else communicating directly between docker and the cluster.  From here on out we will be using YAML files (included in this repository) to describe the deployment, and so you will need to execute these commands from your root project directory.  So go ahead and change to the root directory of the repository you just cloned.

## What's "Kubectl" Really Do?

It's a good time to point out that `kubectl` is just a friendly CLI that gives us convenient access to the Kubernets API Server.  When we add or change configuration to Kubernetes with commands like `kubectl run ...`, there are PUT/POST calls being issued to the Kubernetes API Server.  Likewise, once configured, we can retrieve the configuration with an HTTP GET, or `kubectl get ...`.

## Get the Configuration for your Deployment

While the `kubectl ...` CLI commands are useful for your Developer Inner Loop, when it comes time to deploy to Test and Production environments you want something a little more repeatable.  We want the declarative configuration of the `Deployment` we were just exploring in a text based format that we can store it alongside our application code in Git.  Using `kubectl` again, we can query that configuration back out of the Kubernetes cluster.

So there's a lot of good configuration already in our cluster for the `Deployment` we have. So, let's check it out by getting the deployment information out as YAML:

```bash
kubectl get deployment cn-demo -o yaml
```
That command semantically says "get me the configuration of the Deployment named cn-demo and format it as YAML.
After running this command, you should see a bunch of information about your deployment. 

It'd be a shame to lose it, so let's get it out and store it in a file.
From the root of the repo you cloned (at the same level as the app.py and Dockerfile), run the following command:

```bash
kubectl get deployment cn-demo -o yaml > k8s/lab/deployment.yaml
```
This says "get me the configuration of the Deployment and write it to the file k8s/lab/deployment.yaml".  Go ahead and open that file in VS Code(or a text editor) and look around. The path is relative to the root of the repo you cloned. So, you should navigate to the "k8s" folder then "lab" to find it.  
Do you see the familiar values you defined with the `kubectl` CLI earlier for the `cloud-native-demo` image and the `replicas` count?  There's a lot of other data in there too, some relevant, some not.

Using `kubectl get ...` is a common technique to bootstrap your configuration without needing to remember all the structure of the YAML document, which in turn is really just the API specification for Kubernetes Deployments.  We're going to use a cleaned up version of `deployment.yaml` for our next steps, but just remember how to use `kubectl get ... -o yaml` in the future.  You'll use it often to troubleshoot your Kubernetes configuration.

> Note: In this example we **piped** the results of `kubectl get ...` to a file using the `>` character.  If you leave that part off, the results will be written to your console.  You'll do this often when you just need to see the configuration of a particular Kubernetes object.

Before we continue, let's clean up our current deployment.  Going forward, we'll work with and deploy using variations of `deployment.yaml` contained in this repository.

```bash
kubectl delete deployment cn-demo
```
Now that we have the declarative configuartion of our deployment saved in the deployment.yaml, we can simply recreate it by using the file.  So, let's recreate our deployment, but instead of using our "kubectl create deployment ..." command we will use the deployment.yaml file.  

Use the following command to create the deployment with the file(Remember to be in the root of the repository folder):

```bash
kubectl create -f k8s/lab/deployment.yaml
```

If you run the **kubectl get all** command you should see the created deployment and pods.
Now play with modifying the **replicas:1** under the **spec** section to increase it from 1 to 3. (Don't forget to save it)  
![Replicas In Spec](/images/replicas.PNG)


After updating the replicas, issue this command to delete it:
```bash
kubectl delete -f k8s/lab/deployment.yaml
```

Now recreate it with the kubectl create command:

```bash
kubectl create -f k8s/lab/deployment.yaml
```

After recreating with 3 replicas, run `kubectl get deploy,pods` to see that it updated based on your deployment.yaml changes.


This is just a brief introduction to declarative manifests and theres still a lot more to them, but hopefully you can begin to understand how useful they will be.  You can have a versioned file that spells out exactly how to deploy and configure your application.  Truly infrastructure as code.
