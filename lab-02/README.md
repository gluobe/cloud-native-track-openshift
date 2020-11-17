# Lab 02 - Creating an app using Docker build

In this exercise we will learn how to create an application from a Dockerfile.
OpenShift takes Dockerfile as an input and generates your application Docker
image for you.

## Task 1: Create a project

Export the username environment variable. This to be sure that you are going to
create the correct objects.

```
export USER_NAME=<username>
```

For this lab we are going to create fresh project. Use the following command.

```
oc new-project lab-02-${USER_NAME}
```

## Task 2: Creating an app using a Dockerfile

This time we will use a project that has a Dockerfile in a source code
repository. We will use a simple project on github https://github.com/gluobe/time.
The `rhel` folder from this github project is built starting with rhel7 as the
base image which is described in Dockerfile. Look at the Dockerfile for this
project. It starts off with `registry.access.redhat.com/rhel7` image. It copies
the source code which is a simple `init.sh` file and exposes port `8080`.
Look at the `init.sh` that just displays the current datetime. There is also a
PHP version of the same project available in the php folder if you like to use
that. The php version does exactly the same it has a `time.php` file that
displays the time.

*Docker Build*: When OpenShift finds a Dockerfile in the source, it uses this
Dockerfile as the basis to create a docker image for your application. This
strategy is called `Docker Build` strategy on OpenShift. We’ll see more about it
when we look at the build configuration a couple of steps down the line. Once
OpenShift builds the application’s docker image, it stores that in a local
docker registry. Later it uses this image to deploy an application that runs in
a pod.

Now let’s create an application using this approach. We will run `oc new-app`
command by supplying the git uri as the parameter.

```
oc new-app https://github.com/gluobe/time --context-dir=rhel

---

--> Found Docker image 1d309a6 (6 weeks old) from registry.access.redhat.com for "registry.access.redhat.com/rhel7"

    * An image stream will be created as "rhel7:latest" that will track the source image
    * A Docker build using source code from https://github.com/RedHatWorkshops/time will be created
      * The resulting image will be pushed to image stream "time:latest"
      * Every time "rhel7:latest" changes a new build will be triggered
    * This image will be deployed in deployment config "time"
    * Port 8080 will be load balanced by service "time"
      * Other containers can access this service through the hostname "time"
    * WARNING: Image "registry.access.redhat.com/rhel7" runs as the 'root' user which may not be permitted by your cluster administrator

--> Creating resources with label app=time ...
    imagestream "rhel7" created
    imagestream "time" created
    buildconfig "time" created
    deploymentconfig "time" created
    service "time" created
--> Success
    Build scheduled, use 'oc logs -f bc/time' to track its progress.
    Run 'oc status' to view your app.
```

You’ll notice that OpenShift created a few things at this point. You will find a
buildconfig, deploymentconfig, service and imagestreams in the above list. The
application is not running yet. It needs to be built and deployed. Within a
minute or so, you will see that OpenShift starts the build.

## Task 3: Build

In the meantime lets have a look at the buildconfig by running the command
shown below.

```
oc get bc time -o yaml

---

apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  annotations:
    openshift.io/generated-by: OpenShiftNewApp
  creationTimestamp: 2019-03-20T03:35:54Z
  labels:
    app: time
  name: time
  namespace: lab-02-user13
  resourceVersion: "392826"
  selfLink: /apis/build.openshift.io/v1/namespaces/lab-02-user13/buildconfigs/time
  uid: 447e257f-4ac1-11e9-bd5e-06684e10da10
spec:
  failedBuildsHistoryLimit: 5
  nodeSelector: null
  output:
    to:
      kind: ImageStreamTag
      name: time:latest
  postCommit: {}
  resources: {}
  runPolicy: Serial
  source:
    contextDir: rhel
    git:
      uri: https://github.com/gluobe/time
    type: Git
  strategy:
    dockerStrategy:
      from:
        kind: ImageStreamTag
        name: httpd-24-rhel7:latest
    type: Docker
  successfulBuildsHistoryLimit: 5
  triggers:
  - github:
      secret: AsEu8vO0PJncZ8jTVu-k
    type: GitHub
  - generic:
      secret: 7fOeaFdVt5d74J7Y5_uC
    type: Generic
  - type: ConfigChange
  - imageChange:
      lastTriggeredImageID: registry.access.redhat.com/rhscl/httpd-24-rhel7@sha256:105818bdeec6490dffbb0d1de97aff7fba7545a10c1583d02f62c374b185b8fd
    type: ImageChange
status:
  lastVersion: 1
```

Note the name of the buildconfig in metadata is set to `time`, the git uri
pointing to the value you gave while creating the application. Also note the
Strategy.type set to `Docker`. This indicates that the build will use the
instructions in this Dockerfile to do the docker build.

Build starts in a minute or so. You can view the list of builds using
`oc get builds` command. You can also start the build using
`oc start-build time` where `time` is the name we noticed in the buildconfig.

```
oc get builds

---

NAME      TYPE      FROM          STATUS     STARTED          DURATION
time-1    Docker    Git@1ec2d66   Complete   19 minutes ago   1m13s
```

Note the name of the build that is running i.e. time-1. We will use that name to
look at the build logs. Run the command as shown below to look at the build
logs. This will run for a few mins. At the end you will notice that the docker
image is successfully created and it will start pushing this to OpenShift’s
internal docker registry.

```
oc logs build/time-1

---

<output ommited>
Successfully built 492e4a3bf772
Pushing image docker-registry.default.svc:5000/mycliproject-user02/time:latest ...
Pushed 0/5 layers, 60% complete
Pushed 1/5 layers, 60% complete
Pushed 2/5 layers, 63% complete
Pushed 3/5 layers, 62% complete
Pushed 4/5 layers, 80% complete
Pushed 5/5 layers, 100% complete
Push successful
```

In the above log note how the image is pushed to the local docker registry. The
registry is running at `docker-registry.default.svc` at port `5000`.

## Task 4: Deployment

Once the image is pushed to the docker registry, OpenShift will trigger a deploy
process. Let us also quickly look at the deployment configuration by running the
following command. Note dc represents deploymentconfig.

```
oc get dc -o yaml

---

apiVersion: v1
items:
- apiVersion: apps.openshift.io/v1
  kind: DeploymentConfig
  metadata:
    annotations:
      openshift.io/generated-by: OpenShiftNewApp
    creationTimestamp: 2019-03-20T03:35:55Z
    generation: 2
    labels:
      app: time
    name: time
    namespace: lab-02-user13
    resourceVersion: "392967"
    selfLink: /apis/apps.openshift.io/v1/namespaces/lab-02-user13/deploymentconfigs/time
    uid: 4486a236-4ac1-11e9-bd5e-06684e10da10
  spec:
    replicas: 1
    revisionHistoryLimit: 10
    selector:
      app: time
      deploymentconfig: time
    strategy:
      activeDeadlineSeconds: 21600
      resources: {}
      rollingParams:
        intervalSeconds: 1
        maxSurge: 25%
        maxUnavailable: 25%
        timeoutSeconds: 600
        updatePeriodSeconds: 1
      type: Rolling
    template:
      metadata:
        annotations:
          openshift.io/generated-by: OpenShiftNewApp
        creationTimestamp: null
        labels:
          app: time
          deploymentconfig: time
      spec:
        containers:
        - image: docker-registry.default.svc:5000/lab-02-user13/time@sha256:9bd6c12db3e97c06e0f154f043377d22a8cc5a0d49d2a8016c818d2b08ff5c02
          imagePullPolicy: Always
          name: time
          ports:
          - containerPort: 8080
            protocol: TCP
          - containerPort: 8443
            protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
        dnsPolicy: ClusterFirst
        restartPolicy: Always
        schedulerName: default-scheduler
        securityContext: {}
        terminationGracePeriodSeconds: 30
    test: false
    triggers:
    - type: ConfigChange
    - imageChangeParams:
        automatic: true
        containerNames:
        - time
        from:
          kind: ImageStreamTag
          name: time:latest
          namespace: lab-02-user13
        lastTriggeredImage: docker-registry.default.svc:5000/lab-02-user13/time@sha256:9bd6c12db3e97c06e0f154f043377d22a8cc5a0d49d2a8016c818d2b08ff5c02
      type: ImageChange
  status:
    availableReplicas: 1
    conditions:
    - lastTransitionTime: 2019-03-20T03:36:24Z
      lastUpdateTime: 2019-03-20T03:36:24Z
      message: Deployment config has minimum availability.
      status: "True"
      type: Available
    - lastTransitionTime: 2019-03-20T03:36:25Z
      lastUpdateTime: 2019-03-20T03:36:25Z
      message: replication controller "time-1" successfully rolled out
      reason: NewReplicationControllerAvailable
      status: "True"
      type: Progressing
    details:
      causes:
      - type: ConfigChange
      message: config change
    latestVersion: 1
    observedGeneration: 2
    readyReplicas: 1
    replicas: 1
    unavailableReplicas: 0
    updatedReplicas: 1
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

Note where the image is picked from. It shows that the deployment picks the
image from the local registry (same ip address and port as in buildconfig) and
the image tag is same as what we built earlier. This means the deployment step
deploys the application image what was built earlier during the build step.

If you get the list of pods, you’ll notice that the application gets deployed
quickly and starts running in its own pod.

```
oc get pods

---

NAME           READY     STATUS      RESTARTS   AGE
time-1-build   0/1       Completed   0          2h
time-1-rqa7c   1/1       Running     0          2h
```

## Task 5: Adding route

This step is very much the same as what we did in the previous exercise. We will
check the service and add a route to expose that service.

```
oc get services

---

NAME      CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
time      172.30.xx.82   <none>        8080/TCP   2h
```

Here we expose the service as a route.

```
oc expose service time

---

route.route.openshift.io/time exposed
```

And then we check the route exposed.

```
oc get routes

---

NAME      HOST/PORT                                            PATH      SERVICES   PORT       TERMINATION   WILDCARD
time      time-lab-02-${USER_NAME}.apps.openshift-workshop.gluo.io             time       8080-tcp                 None
```

## Task 6: Running the application

Now run the application by using the route you provided in the previous step.
You can use either curl or your browser. The application displays a success
message.

```
curl time-lab-02-<USER_NAME>.apps.openshift-workshop.gluo.io

Congratulations you just deployed your app by using a Docker build strategy!
```

Congratulations!! In this exercise you have learnt how to create, build and
deploy an application using OpenShift’s `Docker Build strategy`.

## Task 7 : Delete your project

Delete the project with the following command.

```
oc delete project lab-02-${USER_NAME}

---

project.project.openshift.io "lab-02-user13" deleted
```
