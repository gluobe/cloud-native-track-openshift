# Lab - 09 Rolling back applications

It's also possible to rollback your applications. Imagine that our last commit
contained had a bug. We can easily rollback the deployment with the following 
steps.

## Task 1:  Get the current deployment configs

We will need the name of the deployment config in order to rollback the entire
deployment.

```
oc get dc

NAME            REVISION   DESIRED   CURRENT   TRIGGERED BY
scm-web-hooks   2          1         1         config,image(scm-web-hooks:latest)
```

When we know the name of the deployment config we can use the name to rollback 
the entire application. The output should be something like this.

```
oc rollback dc/scm-web-hooks

deploymentconfig.apps.openshift.io/scm-web-hooks deployment #3 rolled back to scm-web-hooks-1
Warning: the following images triggers were disabled: scm-web-hooks:latest
  You can re-enable them with: oc set triggers dc/scm-web-hooks --auto
```

Check out your deployment by getting the route and browse to the `HOST/PORT`
section of the output.

```
oc get route

NAME            HOST/PORT                                                     PATH      SERVICES        PORT       TERMINATION   WILDCARD
scm-web-hooks   scm-web-hooks-lab-08-user13.apps.openshift-workshop.gluo.io             scm-web-hooks   8080-tcp                 None
```

We succefully rolled back our application. If you for example tweaked the colour
of the deploy you will see that the old color is back.

## Task 2 : Delete your project

You can delete your project in the web console or via the CLI with the following
command.

```
oc delete project lab-08-${USERNAME}
```