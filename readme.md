# ACME App

## Analysis of the problem
ACME corp. has seen the previous improvements to their deployment process from the previous assignment. They have made a containerised version of a similar application and made a CI build to publish it. They are now looking for us developers to start using Kubernetes to deploy the application as the use of kubernetes clusters will provide a great advantage for them.

## Assignment tasks

### B - Create a HELM chart to deploy the application to kubernetes.

The HELM chart was created by executing the command

`helm create acme`
After this, un needed files were removed from the directory.
```$> cd game2048
$> rmdir charts
$> rm -rf templates/*```


To stand up the kubernetes cluster go into the `environment` directory and follow the directions in the make file
