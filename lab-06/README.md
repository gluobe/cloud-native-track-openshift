# Lab - 06 Routes

## Task 1: Create a project

Export the username environment variable. This to be sure that you are going to
create the correct objects.

```
export USERNAME=<username>
```

For this lab we are going to create fresh project. Use the following command.

```
oc new-project lab-06-${USERNAME}
```

Or, of course, create the `lab-06-${USERNAME}` project through the UI.

## Task 2: Deploy an application and create a route

List existing image builder or image streams.

```
oc new-app -S --image-stream=php

Image streams (oc new-app --image-stream=<image-stream> [--code=<source>])
-----
php
  Project: openshift
  Tags:    7.0, 7.1, latest
```

Create an application. We will be using a sample application that displays a 
blue or green rectangle. The sample app can be browsed at
https://github.com/RedHatWorkshops/bluegreen

We will be using an env var in order to change the color of the box; but in 
practice you would use a different branch for each version of the code. (E.g. v1 
branch and v2 branch)

```
oc new-app --image-stream=php --code=https://github.com/RedHatWorkshops/bluegreen --env COLOR=blue --name=blue
```

Now monitor the build.

```
oc get builds

NAME      TYPE      FROM          STATUS    STARTED          DURATION
blue-1    Source    Git@9008f89   Running   20 seconds ago   
```

Using the build name of the recently created application run:

```
oc logs build/blue-1 -f
```

Once the build finishes you should see something similar to (hit `CTRL+x` if 
you do):

```
Pushing image 172.30.1.1:5000/bluegreen-joris/blue:latest ...
Pushed 0/10 layers, 1% complete
Pushed 1/10 layers, 10% complete
Pushed 2/10 layers, 20% complete
Push successful
```

Check the application deployment status.

```
oc get pods

NAME           READY     STATUS      RESTARTS   AGE
blue-1-4pt7t   1/1       Running     0          30s
blue-1-build   0/1       Completed   0          1m
```

Notice that the build pod has exited and you now have a single instance
of the application running under one single pod.

This application displays a blue square. List the service.

```
oc get service

NAME      CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
blue      172.30.52.106   <none>        8080/TCP,8443/TCP   3m
```

Now we need to create a route for the service.

```
oc expose service blue --name=blue-route
```

Test the application. Do the following command an paste the `HOST/PORT` to your
browser.

```
oc get route

NAME         HOST/PORT                                                  PATH      SERVICES   PORT       TERMINATION   WILDCARD
blue-route   blue-route-lab-06-<USERNAME.apps.openshift-workshop.gluo.io          blue       8080-tcp                 None
```

## Task 3: Delete your project

You can delete your project in the web console or via the CLI with the following
command.

```
oc delete project lab-06-${USERNAME}
```