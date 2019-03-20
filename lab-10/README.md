# Lab - 10 Using Config Map

## Task 1: Create a project

Export the username environment variable. This to be sure that you are going to
create the correct objects.

```
export USERNAME=<username>
```

For this lab we are going to create fresh project. Use the following command.

```
oc new-project lab-10-${USERNAME}
```

Or, of course, create the `lab-10-${USERNAME}` project through the UI.

## Task 2 : Create an application with configmap

With a configmap we are able to easily pass environment variables to the 
deployment we are creating.

Create a file `ui.properties` with the following content:

```
color=green
```

This will pass an environment variable to the pods that we will create with our
deployment. We are now able to create a configmap with this file. We are also
going to string using the `-from-literal` option at the command line:

```
oc create configmap config --from-literal=message='Hello World!' --from-file=ui.properties
```

Verify that the configmap has been created succesfully:

```
oc get configmap/config -o yaml

apiVersion: v1
data:
  message: Hello World!
  ui.properties: |
    color=green
kind: ConfigMap
metadata:
  creationTimestamp: 2019-03-20T06:58:44Z
  name: config
  namespace: lab-10-user13
  resourceVersion: "424197"
  selfLink: /api/v1/namespaces/lab-10-user13/configmaps/config
  uid: 9a12983b-4add-11e9-b86c-0a5a33ba2bc0
```

Instead of manually creating the application we are going to use some json files 
this time (just to show that you could also use json instead of yaml).

Get the `node-app-build.json` and the `node-app-deployment.json` from this repo
and apply them with the following commands.

```
oc apply -f node-app-deployment.json

imagestream.image.openshift.io/node-app created
deploymentconfig.apps.openshift.io/node-app created
service/node-app created
route.route.openshift.io/node-app created
```

```
oc apply -f node-app-build.json

buildconfig.build.openshift.io/node-app created
```

After a while you will be able to browse to the endpoint shown on the dashboard.
You will see that we will get a green background with the `Hello World!` 
message.

```
oc get route

NAME       HOST/PORT                                                PATH      SERVICES   PORT       TERMINATION   WILDCARD
node-app   node-app-lab-10-<USERNAME>.apps.openshift-workshop.gluo.io         node-app   8080-tcp                 None
```

Open your browser and go to: http://node-app-lab-10-<USERNAME>.apps.openshift-workshop.gluo.io

## Task 3 : Edit the configmap

```
oc edit configmap config

# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  message: Hello World!
  ui.properties: |
    color=green
kind: ConfigMap
metadata:
  creationTimestamp: 2019-03-20T06:58:44Z
  name: config
  namespace: lab-10-user13
  resourceVersion: "425701"
  selfLink: /api/v1/namespaces/lab-10-user13/configmaps/config
  uid: 9a12983b-4add-11e9-b86c-0a5a33ba2bc0
```

Edit the color to yellow, a new app should be deployed with the new config. This 
might take a while, so you will probably need to refresh a couple of times 
before the change goes live. 

## Task 4 : Delete your project

You can delete your project in the web console or via the CLI with the following
command.

```
oc delete project lab-10-${USERNAME}
```