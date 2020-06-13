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

I have also created a service manifest, in which I define the name of the load balancer and what ports it can run on.

Once i had manually created all of the required files in the helm chart, I added the Package helm job to my pipeline. This job packages up the helm chart, then moves it to our artifact directory.
```
      - run:
          name: Package helm
          command: | 
            cd helm 
            helm package acme --app-version ${CIRCLE_SHA1}
            mv acme-0.1.0.tgz ../artifacts/acme.tgz 
```

In order to pass in the Database endpoint, When deploying the infastructure I ensure to output the endpoint from terraform into a text file which is stored in the artifacts directory. Then, when deployying the application through helm, I append the value of this text file as the dbhost varriable.
![endpoint](https://i.imgur.com/xCvFGmZ.png)

## C - Deploy the application into a non-production environment
Here we create a deploy-test job which will include the following run commands for each part of this criteria. It defines the docker image we wish to use, and also attatches it to our workspace.
```
  deploy-test:
    docker:
      - image: cimg/base:2020.01
    environment:
      ENV: test
    steps:
      - attach_workspace:
          at: ./
```


### create a namespace for the application
This was done by creating a job to create the applications namespaces as follows:
```
      - run:
          name: create namespaces
          command: |
             kubectl get namespace test || kubectl create namespace test
             kubectl get namespace prod || kubectl create namespace prod
```
Both the test and prod namespaces are made here. The left half of the command checks if it exists, if not it will create the namespace.

### Deploy a database to back the application
This is done during the deploy infra step. Where running the make init and make up scripts, we create and RDS cluster and export the endpoint to our artifacts text file.
```
      - run:
          name: deploy infra
          command: |
            cd artifacts/infra
            make init
            make up
            terraform output endpoint > ../dbendpoint.txt
```

### Deploy the HELM chart to the Kubernetes cluster
In order to deploy the HELM chart to the Kubernetes cluster we run the job as shown below. We use helm upgrade with the -i flag so that the helm chart will be installed if it is not alredy. We also specify that we want to use the test namespace, and later pass in the image and db endpoint that we saved earlier.
```
     - run:
          name: deploy app
          command: | 
            helm upgrade acme artifacts/acme.tgz -i -n test --wait --set image=$(cat artifacts/image.txt),dbhost=$(cat artifacts/dbendpoint.txt)
```
### Run Database migration script against the database
This is done after applying the helm chart, with the code snippet as below. It simply executes the db migrate script provided with the assignment. I am unsure why i need the sleep command in there, but it will break without it.
```
      - run:
          name: upgrade database
          command: |
            sleep 10s
            kubectl exec deployment/acme -n test -- env NODE_ENV=production node_modules/.bin/sequelize db:migrate
```

### Viewing the test environment
After all of this, the application can be found be pasting the load balancer address into a web browser, as shown bellow:
![deployed](https://i.imgur.com/ATxzEEW.png)

To stand up the kubernetes cluster go into the `environment` directory and follow the directions in the make file


