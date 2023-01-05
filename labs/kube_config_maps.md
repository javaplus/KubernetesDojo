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

We are going to cover the first two in this tutorial, and leave the remainder as an exercise for you!



