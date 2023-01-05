# ~~ Config Maps ~~

We've seen how easy it is to define environment variables used by our application using the ```env:``` section in our Pod deployment yaml.  Kubernetes also allows you to decouple your configuration from your Pod declaration using ConfigMaps.

## Creating ConfigMaps from Literals

Like most Kubernetes commands, there are multiple ways to create ConfigMaps.  Lets start by creating a simple ConfigMap using command line parameters.

From any folder, execute the following kubectl command:

```bash
# Create ConfigMap from Literal
kubectl create configmap codemash-config --from-literal=cm.user_defined_1=my-cm-value1 --from-literal=cm.user_defined_2=my-cm-value2
```

This command is defining two name value pairs to be stored in the configmap.  One should be ```cm.user_defined_1``` with a value of ```my-cm-value1``` and the other should be ```cm.user_defined_2``` with a value of ```my-cm-value2```

Once the configmap is created, lets print the values.  Note the contents of the ```data:``` section

To see the contents of the config map in Kubernetes, run the following kubectl command:
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

## Creating ConfigMaps from Files

The ```from-literal``` sytanx is convenient for one or two name-value pairs, but quickly becomes cubersome with many config parameters.  To resolve this problem, Kuberenetes also provides methods for creating config-maps from files.   We'll look at that next.

### Create ConfigMaps from a single file

For this scenario, we will use the file ```app.properties```, located in the ```/k8s/app-configmaps``` directory.  If you view that file, you will notice its a typical properties file with name value pairs.

Now From the  ```/k8s/app-configmaps``` directory, run the following command, and then print the contents of the configmap

```bash
# Create ConfigMap from app.properties
kubectl create configmap app-config --from-file=app.properties

# Print the contents of the app-config configmap as yaml
kubectl get configmaps app-config -o yaml
```

Your output should be similar to

```
c:\>kubectl get configmaps app-config -o yaml
apiVersion: v1
data:
  app.properties: "USER_DEFINED_2=my-user-define-value-2\r\nUSER_DEFINED_3=my-user-define-value-3"
kind: ConfigMap
metadata:
  creationTimestamp: "2023-01-05T14:40:34Z"
  name: app-config
  namespace: default
  resourceVersion: "26039"
  uid: 87493ed4-919e-48e7-913f-6e3f2ed7afda
```

Notice, the value under the ```data``` section.   The name of the file has been inserted as the key that must be used when referencing the other config maps, which we will see shortly. Also note that the entire contents of the file are stored in the config map, including comments and CRLF.

### Creating a ConfigMap from an Environment File

Kuberenetes also recognizes a special type of file, called an env-file, that has the following syntax rules:

```bash
# Env-files contain a list of environment variables.
# These syntax rules apply:
#   Each line in an env file has to be in VAR=VAL format.
#   Lines beginning with # (i.e. comments) are ignored.
#   Blank lines are ignored.
#   There is no special handling of quotation marks (i.e. they will be part of the ConfigMap value)).
```

Many times this is a better approach, because we get the benefit of more control over the contents of our values in the config map.

Follow these steps to create an env-file:

1. Create a new file called ```redis.properties``` in the ```\k8s\app-configmaps``` folder 
2. Follow the syntax fules above to store the following config parameters in your file, including a comment line at the top of the file, and an empty line between the parameters:

    |Parameter|Value|
    |---|--|
    |REDIS_HOST|redis-test|
    |REDIS_PORT|"6379 |
    

>TODO If you're struggling with creating this file, see xxxxx

Once the file has been created, lets create a configmap from it using the following command, and then print the contents.
```bash
# Create configmap code-mash-env from env-file
kubectl create configmap redis-env --from-env-file=redis.properties

# Print the contents of the app-config configmap as yaml
kubectl get configmaps redis-env -o yaml
```
Your output should be similar to the following:
```
c:\>kubectl get configmaps redis-env -o yaml
apiVersion: v1
data:
  REDIS_HOST: redis-test
  REDIS_PORT: '"6379"'
kind: ConfigMap
metadata:
  creationTimestamp: "2023-01-05T15:08:23Z"
  name: redis-env
  namespace: default
  resourceVersion: "28122"
  uid: 8ad2adcf-1096-4970-ae6f-4c9eaed59b9c
```

Notice the ```data:``` section only contains the expected name value pairs.

## Referencing ConfigMaps in Pods
Now that we've created configmaps in a vareity of ways, lets see how we reference those values in our Pod deployment.

From the official [Kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/configmap/#configmaps-and-pods):
> There are four different ways that you can use a ConfigMap to configure a container inside a Pod:
>1. Inside a container command and args
>2. Environment variables for a container
>3. Add a file in read-only volume, for the application to read
>4. Write code to run inside the Pod that uses the Kubernetes API to read a ConfigMap

We are going to cover the first two in this lab, and leave the remainder as an exercise for you!

### Referencing Container Environment Variables using ConfigMaps

The first method for referencing a configmap variable is to use the ```valueFrom:``` block and specifying the ```configMapKeyRef```.  

Open the \k8s\app-configmaps\deployment-base.yaml file and add the following section below name of the name of the container:
```yaml
      env:
        - name: USER_DEFINED_1   
          valueFrom:   
            configMapKeyRef:
                 # The ConfigMap containing the value you want to assign to USER_DEFINED_1   
              name: codemash-config   
              # Specify the key associated with the value   
              key: cm.user_defined_1
```

Just like we did in the Environment Vars lab,  lets apply this ```Deployment```, expose the ```Deployment```, and port-forward to the service:

```bash
# Apply the deployment
kubectl apply -f k8s/app-configmaps/deployment-base.yaml

# Expose the deployment on port 5000
kubectl expose deployment cn-demo --port 5000 --target-port 5000

# Port forward to the service on port 5000
kubectl port-forward svc/cn-demo 5000
```
Notice how we referenced the config map, then the key in the map to get the value?

Duplicate this approach for the USER_DEFINED_2 variable using the ```codemash-config`` map you created earlier.

>If you need help, look at the XXXX file

While this is useful, you can see where having many values in a single config map force us to duplicate many entries.   Fortunately, Kubernetes provides us with the ```envFrom:``` shortcut to create environment variables from a single config map without the need to specify each one.

Add the following block below the ```env``` section you just defined:
```yaml
envFrom:
- configMapRef:
    name: redis-env
```
Now lets add the ```command``` section to reference these varialbes using the ```$(VARIABLE)``` syntax, just as we did in the Environment variables lab:

Add the following block below the envFrom: section
```yaml
        command:
          - sh
          - -c
          - |
            python app.py --redis-host $(REDIS_HOST) 
```
Now apply the ```deployment``` and run the port forward command:
```bash
# Apply the deployment
kubectl apply -f k8s/app-configmaps/deployment-base.yaml

# Port forward to the service on port 5000
kubectl port-forward svc/cn-demo 5000
```
Now when you hit http://localhost:5000 you should see all of the values listed that were stored in the Config Maps!  Congratulations, you've succesfully extracted your config from your app!

For more information on ConfigMaps, and other ways to use them, check out the following resources:
- [Kubernetes ConfigMap Docs](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [How to Configure a POd to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)


