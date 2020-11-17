# Lab 01 - Creating an app from a Docker image

In this lab you will learn how to create a new project on OpenShift and how to
create an application from an existing docker image.

## Task 1: Create a new project from command line

We will first exporting your username as a variable, that way it will be easy to
just copy/paste the commands below.

```
export USER_NAME=<REPLACE_WITH_YOUR_USER_NAME>
```

Make sure that you have exported your username correctly:

```
echo ${USER_NAME}

---

user04
```

If your username is not echoed correctly, you probably made a mistake with the 
previous `export` command, so try that command again.

The command below creates a new project and is setting a description and display
name for the project.

```
oc new-project lab-01-${USER_NAME} --description="lab-01 - ${USER_NAME}" --display-name="lab-01 - ${USER_NAME}"

---

Now using project "lab-01-user13" on server "https://openshift-workshop.gluo.io:8443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app centos/ruby-25-centos7~https://github.com/sclorg/ruby-ex.git

to build a new example application in Ruby.
```

Upon project creation, OpenShift will automatically switch to the newly created
project/namespace. If you wish to view the list of projects, run the following
command:

```
oc get projects
```

If you have more than one project, you can switch to a different one by issuing
`oc project <project name>`. Although you don’t want to do it now.

You can also check the status of the project by running the following command.
It says that the project is currently not running anything.

```
oc status

In project lab-01 - <USER_NAME> (lab-01-<USER_NAME>) on server https://openshift-workshop.gluo.io:8443

You have no services, deployment configs, or build configs.
Run 'oc new-app' to create an application.
```

## Task 2: Create an app from a Docker image

Next we will create an application inside the above project using an existing
docker image. We will be using a very simple Docker image from Docker Hub that
just says 'Welcome to OpenShift V3. Let us just use that for this exercise.

First create a new application using the docker image using the `oc new-app`
command as shown below:

```
oc new-app gluobe/welcome-php --name=welcome

---

--> Found Docker image 616b725 (2 years old) from Docker Hub for "gluobe/welcome-php"

    chx/welcome-php-1:b68a2d86
    --------------------------
    Platform for building and running PHP 5.6 applications

    Tags: builder, php, php56, rh-php56

    * An image stream tag will be created as "welcome:latest" that will track this image
    * This image will be deployed in deployment config "welcome"
    * Port 8080/tcp will be load balanced by service "welcome"
      * Other containers can access this service through the hostname "welcome"

--> Creating resources ...
    imagestream.image.openshift.io "welcome" created
    deploymentconfig.apps.openshift.io "welcome" created
    service "welcome" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/welcome'
    Run 'oc status' to view your app.
```

The above command uses the docker image to deploy a docker container in a pod.
If you quickly run `oc get pods` you will notice that a deployer pod runs and it
starts an application pod as shown below.

```
oc get pods

---

NAME               READY     STATUS    RESTARTS   AGE
welcome-1-deploy   1/1       Running   0          1m
welcome-1-dkyyq    0/1       Pending   0          0s
```

In the above example `welcome-1-deploy` is the deployer pod and the other one is
the actual application pod. In a little while the deployer pod will succeed and
the application pod will change for `Pending` to `Running` status.

```
oc get pods

---

NAME              READY     STATUS    RESTARTS   AGE
welcome-1-dkyyq   1/1       Running   0          56s
```

## Task 3: Add a route for your app

OpenShift also spins up a service for this application. Run the following
command to view the list of services in the project (you can also use
`oc get svc` shorthand).

```
oc get services

---

NAME      CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
welcome   172.30.77.93   <none>        8080/TCP   2m
```

You will notice the `welcome` service was created for this project.

However, there is no route for this application yet. So you cannot access this
application from outside.

Now add a route to the service with the following command. `oc expose` command
will allow you to expose your service to the world so that you can access it
from the browser.

*Note*: In this example, I am using a domain name of
`apps.openshift-workshop.gluo.io`.

```
oc expose service welcome --name=welcome

---

route.route.openshift.io/welcome exposed
```

With the `oc get routes` commad very that the route has been created:

```
oc get routes

---

NAME      HOST/PORT                                               PATH      SERVICES   PORT       TERMINATION   WILDCARD
welcome   welcome-lab-01-<USER_NAME>.apps.openshift-workshop.gluo.io             welcome    8080-tcp                 None
```

## Task 4: Test your app

From the `oc get routes` copy the host and surf to it from your browser:
http://welcome-lab-01-<USER_NAME>.apps.openshift-workshop.gluo.io

Voila!! you created your first application using an existing docker image on
OpenShift.

## Task 5: Clean up

Run the `oc get all` command to view all the components that were created in
your project.

```
oc get all

---

NAME                  READY     STATUS    RESTARTS   AGE
pod/welcome-1-8gwxj   1/1       Running   0          4m

NAME                              DESIRED   CURRENT   READY     AGE
replicationcontroller/welcome-1   1         1         1         4m

NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/welcome   ClusterIP   172.30.136.19   <none>        8080/TCP   4m

NAME                                         REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfig.apps.openshift.io/welcome   1          1         1         config,image(welcome:latest)

NAME                                     DOCKER REPO                                             TAGS      UPDATED
imagestream.image.openshift.io/welcome   docker-registry.default.svc:5000/labs-trescst/welcome   latest    4 minutes ago

NAME                               HOST/PORT                                                       PATH      SERVICES   PORT       TERMINATION   WILDCARD
route.route.openshift.io/welcome   welcome-labs-trescst.7e14.starter-us-west-2.openshiftapps.com             welcome    8080-tcp                 None
```

Now you can delete all these components by running one command.

```
oc delete all --all

pod "welcome-1-8gwxj" deleted
replicationcontroller "welcome-1" deleted
service "welcome" deleted
deploymentconfig.apps.openshift.io "welcome" deleted
imagestream.image.openshift.io "welcome" deleted
route.route.openshift.io "welcome" deleted
```

You will notice that it has deleted the imagestream for the application, the
deploymentconfig, the service and the route.

You can run `oc get all` again to make sure the project is empty.

```
oc get all

---

No resources found.
```

Congratulations!! You now know how to create a project, an application using an
external docker image and navigate around. Get ready for more fun stuff!

Delete the project with the following command.

```
oc delete project lab-01-${USER_NAME}

---

project.project.openshift.io "lab-01-user13" deleted
```
