# ~~ Config Maps ~~

We've seen how easy it is to define environment variables used by our application using the ```env:``` clause in our Pod declaration.  Kubernetes also allows you to decouple your configuration from your Pod declaration using ConfigMaps.

## Creating ConfigMaps from Literals

Like most Kubernetes commands, there are multiple ways to create ConfigMaps.  Lets start by creating a simple ConfigMap using command line parameters.

```bash
# Create ConfigMap from Literal
kubectl create configmap codemash-config --from-literal=cm.location=kalari --from-literal=cm.year=2023
```

This command is defining two name value pairs to be stored in the configmap.  One should be ```cm.location``` with a value of ```kalahar``` and the other should be ```cm.year``` with a value of ```2023```

Once the configmap is created, lets print the values.  Note the contents of the ```data:``` section

```bash
# Print the contents of the codemash-config configmap as yaml
kubectl get configmaps codemash-config -o yaml
```

Your output should look very similar to the following:

```bash
<FILL ME IN>
```

## Creating ConfigMaps from Files

The ```from-literal``` sytanx is convenient for one or two name-value pairs, but quickly becomes cubersome with many config parameters.  To resolve this problem, Kuberenetes also provides methods for creating config-maps from files.   We'll look at that next.

### Create ConfigMaps from a single file

For this scenario, we will use the file app.properties, located in the /WHAT/DIR directory

From the /WHAT/DIR directory, run the following command, and then print the contents of the configmap

```bash
# Create ConfigMap from app.properties
kubectl create configmap app-config --from-file=./props/app.properties

# Print the contents of the app-config configmap as yaml
kubectl describe configmaps app-config -o yaml
```

Your output should be similar to

```
MORE OUTPUT
```

Notice, the value under the ```Data``` section.   The name of the file, has been inserted as the key that must be used when referencing the other config maps, which we will see shortly.

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

This means we can create a properties file with our configuration and then create a config-map directly from it.

Follow these steps to create an env-file:

1. Create a new file called ```codemash.properties``` 
2. Follow the syntax fules above to store the following config parameters in your file, including a comment line at the top of the file:

    |Parameter|Value|
    |---|--|
    |title|"Kubernetes Dojo"|
    |type| Pre-compileter|
    |capacity|25|

If you're struggling with creating this file, see xxxxx

Once the file has been created, lets create a configmap from it using the following command, and then print the contents.
```bash
# Create configmap code-mash-env from env-file
kubectl create config-map codemash-env --from-env-file=codemash.properties

# Print the contents of the app-config configmap as yaml
kubectl describe configmaps codemash-env -o yaml
```
Your output should be similar to the following:
```
OUTPUT
```

## Referencing ConfigMaps in Pods
Now that we've created configmaps in a vareity of ways, lets see how we reference those values in our Pod deployment.



