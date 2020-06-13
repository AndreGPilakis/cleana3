# ACME App

## Analysis of the problem
ACME corp. has seen the previous improvements to their deployment process from the previous assignment. They have made a containerised version of a similar application and made a CI build to publish it. They are now looking for us developers to start using Kubernetes to deploy the application as the use of kubernetes clusters will provide a great advantage for them.

## Assignment tasks

Before running the pipeline, I initially cloned the project locally and ran the make and make kube files to set up the infustructure. This can be found by navigating to `/environment` and then running `make up`, and finally `make kube-up`.

### B - Create a HELM chart to deploy the application to kubernetes.

The HELM chart was created by executing the command

`helm create acme`
After this, un needed files were removed from the directory.
```
$> cd game2048
$> rmdir charts
$> rm -rf templates/*
```

After this, I created a Deployment manifest for the application as shown below:
![Deployment](https://i.imgur.com/TIAvbrw.png)

Inside this Deployment file I specify many aspects of the deployment. Notably, we are accessing our values file to access the variables for the replicaCount, The image name that is stored in the values file, aswell as all of the database credentials.

To stand up the kubernetes cluster go into the `environment` directory and follow the directions in the make file
