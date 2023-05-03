# ~~ Environment Variables ~~

## Clean up

Before starting this lab, clean up any deployments still running.
Run the following command to delete the deployment we were working with last time.

```bash
kubectl delete -f k8s/lab/deployment.yaml
```

You could delete the deployment without using the deployment.yaml, but using the yaml to delete is less error prone.  
The "manual" way is to just use the delete command with the resource type and resource name like this:

```bash
kubectl delete deploy/cn-demo
```

Run the `kubectl get deploy,pods` to ensure that everything is gone or terminating.


## Redeploy the baseline

Let's suppose that one of the requirements for your app was to adhere to 12 factor apps methodology and therefore it must be configurable via Environment Variables.  Luckily our cn-demo app was built this way and we're now going to put that to the test.  Our first steps will be to redeploy the application, now using the declarative YAML in the `k8s/app-envvars` directory of this project.  To do that, we're going to use the `kubectl apply ...` command.  Be sure to run this command from the root of the repo (same place as last lab).

```bash
kubectl apply -f k8s/app-envvars/deployment-base.yaml
```

If you run your `kubectl get deploy,pods`, you should see a single Pod running again.  Go ahead and take a look at the `k8s/app-envvars/deployment-base.yaml` file to get familiar with it.  It's a slimmed down version of the `Deployment` YAML you got in the previous step with `kubectl get ... -oyaml`.

## Connect to the Container

We never actually tested the application in the last step, but lets do that now by exposing the `Pod` as a `Service` and using the `kubectl port-forward` command to connect to it.  A Kubernetes `Service` is a lengthy topic, and more can be [read here](https://kubernetes.io/docs/concepts/services-networking/service/).  We'll be focusing on creating what's known as a ClusterIP Service, which means there will be a single addressable endpoint available in the cluster to access the Pod(s) hosting our application.  First thing is first, let's expose the `Service` using `kubectl expose ...`.

```bash
kubectl expose deployment cn-demo --port 5000 --target-port 5000
```

This command says "Create a new ClusterIP Service that listens on port 5000 and targets port 5000 of all Pods that are a part of the cn-demo Deployment".  Next lets connect to this newly created `Service`.  The `kubectl port-forward` command will expose a port on our local machine that connects to ports exposed on Pods.  We need to do the `port-forward` command here because the Pod is only accessible inside the cluster right now.  

```bash
kubectl port-forward svc/cn-demo 5000
```

> Earlier we said that port-forward connects to a Pod, but here we're port forwarding to a Service.  What???  In this case, think of the Service as an alias to one to many Pods that it load balances across.  Normally when hitting the service, the traffic would be spread across the different pods, but when port-forwarding to a service, it just picks a single pod to forward traffic to as long as it's running.  It's a kubectl trick, but just a single Pod will be selected for forwarding, even if there are many replicas of it.

At this point we should be able to access [http://localhost:5000](http://localhost:5000) from our browser.  Give it a try.  You should some JSON output similar to below.  Your value for `host` will vary and it may not be pretty printed (may be all on one line).

```json
{
  "appName": "cloud-native-demo",
  "env": {
    "host": "cn-demo-7df548dcf8-wpr8v",
    "user_defined_1": null,
    "user_defined_2": null,
    "user_defined_3": null
  },
  "redis-host": "localhost",
  "version": "1.0.0"
}
```

Once you're finished, stop the port forwarding command by pressing `CTRL+C`.

## Configure the User Defined variables

> TIP: The completed deployment configuration is in the `k8s/app-envvars/deployment-with-envvars.yaml` file if you need help through this section.

Right now we have a lot of `null` values for the `user_defined_#` variables.  Let's fix that.  In the **`k8s/app-envvars/deployment-base.yaml`** file, find the section that looks like the following:

```yaml
      containers:
      - image: javaplus/cloud-native-demo:1
        imagePullPolicy: IfNotPresent
        name: cn-demo
```

Add a new section directly under `name: cn-demo` with the following.  Indentation matters here.

```yaml
        env:
          - name: USER_DEFINED_1
            value: my-user-define-value-1
```

With just this configuration, **save** and re-apply this `Deployment`.  Kubernetes will see the difference in configuration and redeploy a new `Pod` with the updated configuration.  The old `Pod` will be terminated as it no longer matches the desired state of your deployment (it doesn't have the right configuration anymore).  Assuming you didn't change the filename, run the following and watch your dashboard for how the old and new `Pods` cycle.

```bash
kubectl apply -f k8s/app-envvars/deployment-base.yaml
```

Finally, run the same `port-forward` command you ran earlier and try to access the application in your browser at [http://localhost:5000](http://localhost:5000).  Do you see the updated value for `user_defined_1`?  If you look at the application code in the `app.py` file at the root of the repo you cloned, you can see how the Python code is reading from an Environment Variable to populate that section of the JSON response.

## Fill in the remaining environment variables

Using the same pattern as above, fill in the following environment key-value pairs as environment variables and observe the changes in the running application after redeploying.

```bash
USER_DEFINED_2=my-user-define-value-2
USER_DEFINED_3=my-user-define-value-3
REDIS_HOST=redis-test
REDIS_PORT="6379"
```
The last two may not show up in the response. That's ok!

Previous | Next
--- | ---
[Kubernetes Infrastructure as Code](labs/3_/kube_infra_as_code.md) | [Kubernetes Override Starting Command](labs/5_/kube_override_cmd.md)