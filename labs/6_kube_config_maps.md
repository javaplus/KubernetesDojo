# ~~ ConfigMaps ~~

We've seen how easy it is to define environment variables used by our application using the ```env:``` section in our Pod deployment yaml.  However, according the 12 Factor app specification, we need to be able to change our configuration by environment.  While we could maintain multiple versions of our deployment file, this would make maintenance very difficult.  Kubernetes addresses this problem with ConfigMaps.

[ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/) are special Kubernetes objects that store non-confidential data in key-value pairs.  Pods are able to consume these values in a variety of ways, allowing you to decouple your config from your container image, which meets the [Config](https://12factor.net/config) requirement for a 12 Factor application.

## Creating ConfigMaps from Literals

Like most Kubernetes commands, there are multiple ways to create ConfigMaps.  Lets start by creating a simple ConfigMap using command line parameters.

From any folder, execute the following kubectl command:

```bash
kubectl create configmap codemash-config --from-literal=cm.user_defined_1=my-cm-value1 --from-literal=cm.user_defined_2=my-cm-value2
```

This command is defining two key-value pairs to be stored in a single ConfigMap object called codemash-config.  One pair should consist of the key ```cm.user_defined_1``` with a value of ```my-cm-value1``` and the other pair should consist of the key```cm.user_defined_2``` with a value of ```my-cm-value2```

Once the configmap is created, lets print the values.  
```bash
kubectl get configmaps codemash-config -o yaml
```

Your output should look very similar to the following:

```bash
c:\>kubectl get cm codemash-config -o yaml
apiVersion: v1
data:
  cm.user_defined_1: my-cm-value1
  cm.user_defined_2: my-cm-value2
kind: ConfigMap
metadata:
  creationTimestamp: "2023-01-05T14:24:35Z"
  name: codemash-config
  namespace: default
  resourceVersion: "24851"
  uid: 923111b5-76fd-4622-b312-bc58ffa08d29
```
Note the contents of the ```data:``` section.  Each key-value pair is now stored in the ConfigMap, to be referenced by the container.

## Creating ConfigMaps from Files

The ```--from-literal``` sytanx is convenient for one or two key-value pairs, but quickly becomes cubersome with many config parameters.  To resolve this problem, Kuberenetes also provides methods for creating ConfigMaps from files.   We'll look at some of these methods next.

### Creating a ConfigMap from an Environment File

For this scenario, we will use the file ```app.properties```, located in the ```/k8s/app-configmaps``` directory.  An env-file must follow the following set of syntax rules:
```bash
# Env-files contain a list of environment variables.
# These syntax rules apply:
#   Each line in an env file has to be in VAR=VAL format.
#   Lines beginning with # (i.e. comments) are ignored.
#   Blank lines are ignored.
#   There is no special handling of quotation marks (i.e. they will be part of the ConfigMap value)).
```

Now From the root of this repository, run the following command:

```bash
kubectl create configmap app-config --from-env-file=./k8s/app-configmaps/app.properties
```

Next, print the contents of the app-config configmap as yaml:
```bash
kubectl get configmaps app-config -o yaml
```

Your output should be similar to

```
C:\>kubectl get configmaps app-config -o yaml
apiVersion: v1
data:
  cm.user_defined_1: my-user-defined-value-1
  cm.user_defined_2: my-user-defined-value-2
  cm.user_defined_3: my-user-defined-value-3
kind: ConfigMap
metadata:
  creationTimestamp: "2023-01-06T15:16:51Z"
  name: app-config
  namespace: default
  resourceVersion: "60666"
  uid: 0a7b8010-c6cc-4bfa-9af5-bfbc8da7344f
```

Notice the values under the ```data:``` section.   Each of the three key-value pairs are now available in the app-config ConfigMap to be referenced by a container, which we will do shortly.

## Referencing ConfigMaps in Pods
Now that we've created ConfigMaps in a variety of ways, lets see how we can reference those values in our Pod deployment yaml.

From the official [Kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/configmap/#configmaps-and-pods), there are four different ways that you can use a ConfigMap:
>1. Inside a container command and args
>2. Environment variables for a container
>3. Add a file in read-only volume, for the application to read
>4. Write code to run inside the Pod that uses the Kubernetes API to read a ConfigMap

We are going to cover the first two in this lab, and leave the remainder as an exercise for you!

### Referencing Container Environment Variables using ConfigMaps

The first method for referencing a configmap variable is to use the ```valueFrom:``` block and specifying the ```configMapKeyRef```.  

Open the ```\k8s\app-configmaps\deployment-base.yaml``` file and add the following section below name of the name of the container:
```yaml
      env:
        - name: USER_DEFINED_1   
          valueFrom:   
            configMapKeyRef:
                 # The ConfigMap containing the value you want to assign to USER_DEFINED_1   
              name: app-config   
              # Specify the key associated with the value   
              key: cm.user_defined_1
```

This section is telling Kubernetes to find the ConfigMap called ```app-config```, look up the value for the key ```cm.user_defined_1``` and assign it to the environment variable ```USER_DEFINED_1```

Just like we did in the Environment Vars lab,  lets apply this ```Deployment```, and port-forward to the service:

```bash
kubectl apply -f k8s/app-configmaps/deployment-base.yaml

kubectl port-forward svc/cn-demo 5000
```

Now when you access http://localhost:5000 you should see ```USER_DEFINED_1``` has been defined with the value ```my-user-defined-value-1```.

Duplicate this approach for the USER_DEFINED_2 and USER_DEFINED_3 variables using the ```app-config`` map you created earlier.

>If you need help, look at the k8s/app-configmaps/deployment-configmaps.yaml file

### Using envFrom: to Reference ConfigMaps

While ```valueFrom:``` is useful, you can see where having many values in a single ConfigMap would force us to duplicate many references in the ```Deployment```.   Fortunately, Kubernetes provides us with the ```envFrom:``` shortcut to create environment variables from a single ConfigMap without the need to specify each one.

First, lets create a new ConfigMap from the ```redis.properties``` environment file found in ```k8s/app-configmaps/redis.properties```.  

From the root of the repository, run the following command:
```bash
kubectl create configmap redis-env --from-env-file=./k8s/app-configmaps/redis.properties
```

Verify the contents of the ConfigMap with the following command:
```bash
kubectl get configmaps redis-env -o yaml
```

Now add the following block below the ```env``` section where the USER_DEFINED varialbes were defined in the ```\k8s\app-configmaps\deployment-base.yaml``` file :
```yaml
envFrom:
- configMapRef:
    name: redis-env
```

This block tell Kubernetes to create environment variables from ALL of the key-value pairs in the redis-env ConfigMap.

Now lets add the ```command``` section to reference these variables using the ```$(VARIABLE)``` syntax, just as we did in the Environment variables lab:

Add the following block below the ```envFrom:``` section
```yaml
        command:
          - sh
          - -c
          - |
            python app.py --redis-host $(REDIS_HOST) 
```

Lets test our changes.  Apply the ```deployment``` and run the port forward command:
```bash
kubectl apply -f ./k8s/app-configmaps/deployment-base.yaml

kubectl port-forward svc/cn-demo 5000
```
Now when you hit http://localhost:5000 you should see all of the values listed from the app-config ConfigMap, as well as the ```redis-host``` from the redis.properties file.  

> If you are struggling to get these changes to work, please see [k8s/app-configmaps/deployment-configmaps.yaml](../k8s/app-configmaps/deployment-configmaps.yaml) for a working deployment.yaml

Congratulations, you've succesfully extracted your config from your app using multiple ConfigMaps!

## Resources

For more information on ConfigMaps, and other ways to use them, check out the following resources:
- [Kubernetes ConfigMap Docs](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Declaring and using ConfigMaps to configure a Deployment](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

```bash
# --------------------------------------------------
# FOR REFERENCE - COMMANDS USED IN THIS LAB
# --------------------------------------------------

# Create ConfigMap from Literals
kubectl create configmap codemash-config --from-literal=cm.user_defined_1=my-cm-value1 --from-literal=cm.user_defined_2=my-cm-value2

# Create ConfigMap from a single fileapp.properties
kubectl create configmap app-config --from-file=app.properties

# Create configmap redis-env from env-file redis.properties
kubectl create configmap redis-env --from-env-file=redis.properties

# Print the contents of the redis-env configmap as yaml
kubectl get configmaps redis-env -o yaml

# Apply the deployment
kubectl apply -f k8s/app-configmaps/deployment-base.yaml

# Expose the deployment on port 5000
kubectl expose deployment cn-demo --port 5000 --target-port 5000

# Port forward to the service on port 5000
kubectl port-forward svc/cn-demo 5000
```


Previous | Next
--- | ---
[Kubernetes Override Starting Command](5_kube_override_cmd.md) | [Kubernetes Ingress](7_kube_setup_ingress.md)
