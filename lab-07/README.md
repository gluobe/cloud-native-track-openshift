# Lab - 07 Blue/Green deployments

The purpose of this short lab is to demonstrate how simple it is to
implement Blue-Green Deployments in OpenShift.

## Task 1: Create a project

Export the username environment variable. This to be sure that you are going to
create the correct objects.

```
export USERNAME=<username>
```

For this lab we are going to create fresh project. Use the following command.

```
oc new-project lab-07-${USERNAME}
```

Or, of course, create the `lab-07-${USERNAME}` project through the UI.

## Task 2: Deploy the blue version

List existing image builder or image streams.

```
oc new-app -S --image-stream=php

Image streams (oc new-app --image-stream=<image-stream> [--code=<source>])
-----
php
  Project: openshift
  Tags:    7.0, 7.1, latest
```

Create an application. We will be using a sample application that displays a blue or green
rectangle. The sample app can be browsed at
https://github.com/RedHatWorkshops/bluegreen

We will be using an env var in order to change the color of the box; but
in practice you would use a different branch for each version of the
code. (E.g. v1 branch and v2 branch)

```
oc new-app --image-stream=php --code=https://github.com/RedHatWorkshops/bluegreen --env COLOR=blue --name=blue

--> Found image 4757d9f (13 days old) in image stream "openshift/php" under tag "7.1" for "php"

    Apache 2.4 with PHP 7.1
    -----------------------
    PHP 7.1 available as container is a base platform for building and running various PHP 7.1 applications and frameworks. PHP is an HTML-embedded scripting language. PHP attempts to make it easy for developers to write dynamically generated web pages. PHP also offers built-in database integration for several commercial and non-commercial database management systems, so writing a database-enabled webpage with PHP is fairly simple. The most common use of PHP coding is probably as a replacement for CGI scripts.

    Tags: builder, php, php71, rh-php71

    * The source repository appears to match: php
    * A source build using source code from https://github.com/RedHatWorkshops/bluegreen will be created
      * The resulting image will be pushed to image stream tag "blue:latest"
      * Use 'start-build' to trigger a new build
    * This image will be deployed in deployment config "blue"
    * Ports 8080/tcp, 8443/tcp will be load balanced by service "blue"
      * Other containers can access this service through the hostname "blue"

--> Creating resources ...
    imagestream.image.openshift.io "blue" created
    buildconfig.build.openshift.io "blue" created
    deploymentconfig.apps.openshift.io "blue" created
    service "blue" created
--> Success
    Build scheduled, use 'oc logs -f bc/blue' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/blue'
    Run 'oc status' to view your app.
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

Once the build finishes you should see something similar to:

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
oc expose service blue --name=bluegreen

route.route.openshift.io/bluegreen exposed
```

Test the application. Do the following command an copy/paste the `HOST/PORT` 
valueto into your browser.

```
oc get route

NAME        HOST/PORT                                                 PATH      SERVICES   PORT       TERMINATION   WILDCARD
bluegreen   bluegreen-lab-07-<USERNAME>.apps.openshift-workshop.gluo.io         blue       8080-tcp                 None
```

## Task 3: Deploy the green version

Create a new application the same way as you did above in Part I. Make
sure to name the application as `green` this time.

```
oc new-app --image-stream=php --code=https://github.com/RedHatWorkshops/bluegreen --env COLOR=green --name=green

--> Found image 4757d9f (13 days old) in image stream "openshift/php" under tag "7.1" for "php"

    Apache 2.4 with PHP 7.1
    -----------------------
    PHP 7.1 available as container is a base platform for building and running various PHP 7.1 applications and frameworks. PHP is an HTML-embedded scripting language. PHP attempts to make it easy for developers to write dynamically generated web pages. PHP also offers built-in database integration for several commercial and non-commercial database management systems, so writing a database-enabled webpage with PHP is fairly simple. The most common use of PHP coding is probably as a replacement for CGI scripts.

    Tags: builder, php, php71, rh-php71

    * The source repository appears to match: php
    * A source build using source code from https://github.com/RedHatWorkshops/bluegreen will be created
      * The resulting image will be pushed to image stream tag "green:latest"
      * Use 'start-build' to trigger a new build
    * This image will be deployed in deployment config "green"
    * Ports 8080/tcp, 8443/tcp will be load balanced by service "green"
      * Other containers can access this service through the hostname "green"

--> Creating resources ...
    imagestream.image.openshift.io "green" created
    buildconfig.build.openshift.io "green" created
    deploymentconfig.apps.openshift.io "green" created
    service "green" created
--> Success
    Build scheduled, use 'oc logs -f bc/green' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/green'
    Run 'oc status' to view your app.
```

Wait until the application is built and deployed. You should now see two 
services if you run:

```
oc get service

NAME      CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
blue      172.30.52.106    <none>        8080/TCP,8443/TCP   13m
green     172.30.188.189   <none>        8080/TCP,8443/TCP   14s
```

Edit the previously created route and change the `service` name (from `blue` to 
`green`), all the way at the bottom to the new service that was just created. 
You are essentially still using the FQDN you had previously created. However, 
that route will now point to a different (green) service.  Save the file.

```
oc edit route bluegreen

...

# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  annotations:
    openshift.io/host.generated: "true"
  creationTimestamp: 2019-03-20T06:14:31Z
  labels:
    app: blue
  name: bluegreen
  namespace: lab-07-<USERNAME>
  resourceVersion: "417063"
  selfLink: /apis/route.openshift.io/v1/namespaces/lab-07-<USERNAME>/routes/bluegreen
  uid: 6c8b5ec3-4ad7-11e9-b86c-0a5a33ba2bc0
spec:
  host: bluegreen-lab-07-<USERNAME>.apps.openshift-workshop.gluo.io
  port:
    targetPort: 8080-tcp
  to:
    kind: Service
    name: blue   <<< CHANGE THIS FROM 'blue' to 'green'
    weight: 100
  wildcardPolicy: None
status:
  ingress:
  - conditions:
    - lastTransitionTime: 2019-03-20T06:14:31Z
      status: "True"
      type: Admitted
    host: bluegreen-lab-07-<USERNAME>.apps.openshift-workshop.gluo.io
    routerName: router
    wildcardPolicy: None

...

route.route.openshift.io/bluegreen edited
```

Now we are going to test the application again.

```
oc get route

NAME        HOST/PORT                                                 PATH      SERVICES   PORT       TERMINATION   WILDCARD
bluegreen   bluegreen-lab-07-<USERNAME>.apps.openshift-workshop.gluo.io         green      8080-tcp                 None
```

Copy the `HOST/PORT` section of the output in your browser and check out the green
deployment.

## Task 4: Route traffic to both services

First we need to edit the route. Using the lefthand side navigation; click on
`Applications -> Routes`. This will bring you to the `Route` overview page.

![bg-routes-page](../images/bg-routes-page.png "bg-routes-page")

Here, click on the  `bluegreen` route. The page after will display the current 
configuration. On the upper right hand side, click on `Actions -> Edit`. You 
should see a page similar to this one.

![bg-edit-route](../images/bg-edit-route.png "bg-edit-route")

Next, tick on `Split traffic across multiple services`

![bg-slipt-traffic](../images/bg-slipt-traffic.png "bg-slipt-traffic")

Here, set the weight to 50% on blue and 50% on green. This will make it to where 
half the traffic will go to the green application and half to the blue 
application.

![bg-5050-split](../images/bg-5050-split.png "bg-5050-split")

Once you click on `Save`; you should see this on the Route Overview page.

![bg-route-split-overview](../images/bg-route-split-overview.png "bg-route-split-overview")

You are able to test your settings now. If you try and visit your application; 
you'll notice it won't "switch" over to the other application. This is because 
the default behavior is:

* Sticky Session on the Router
* Session Cookie set on the router

To get "true" round robin; annotate your route with the following

```
oc annotate route/bluegreen haproxy.router.openshift.io/balance=roundrobin
oc annotate route/bluegreen haproxy.router.openshift.io/disable_cookies=true
```

Make sure to clear your cookies after issuing the above commands. You wil notice 
that as you are the only visiting the page it might require a couple of tries 
before you will see the other color.

## Task 5 : Delete your project

You can delete your project in the web console or via the CLI with the following
command.

```
oc delete project lab-07-${USERNAME}

project.project.openshift.io "lab-07-user13" deleted
```