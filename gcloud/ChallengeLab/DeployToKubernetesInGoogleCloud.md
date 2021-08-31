# Challenge scenario

You have just completed training on containers and their creation and management and now you need to demonstrate to the Jooli Inc. development team your new skills. You have to help with some of their initial work on a new project around an application environment utilizing Kubernetes. Some of the work was already done for you, but other parts require your expert skills.
You are expected to create container images, store the images in a repository, and configure a Jenkins CI/CD pipeline to automate the build for the product. Your know that Kurt, your supervisor, will ask you to complete these tasks:
- Create a Docker image and store the Dockerfile.
- Test the created Docker image.
- Push the Docker image into the Container Repository.
- Use the image to create and expose a deployment in Kubernetes
- Update the image and push a change to the deployment.
- Create a pipeline in Jenkins to deploy a new version of your image when the source code changes.
Some Jooli Inc. standards you should follow:
- Create all resources in the us-east1 region and us-east1-b zone, unless otherwise directed.
- Use the project VPCs.
- Naming is normally team-resource, e.g. an instance could be named kraken-webserver1.
- Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share: unless directed, use n1-standard-1.
Your challenge
As soon as you sit down at your desk and open your new laptop you receive the following request to complete these tasks. Good luck!
Do not wait for the lab to provision! You can work through tasks 1-3 before you need the provisioning to be finished. Just ensure the workstation exists before starting Task 1.


## Task 1: Create a Docker image and store the Dockerfile
Open Cloud Shell and run source <(gsutil cat gs://cloud-training/gsp318/marking/setup_marking.sh). This command will install marking scripts you can use to help check your progress.
Use Cloud Shell to clone the valkyrie-app source code repository (it is in your project).
The app source code is in valkyrie-app/source. Create valkyrie-app/Dockerfile and add the configuration below.
```
FROM golang:1.10
WORKDIR /go/src/app
COPY source .
RUN go install -v
ENTRYPOINT ["app","-single=true","-port=8080"]
```

Use valkyrie-app/Dockerfile to create a Docker image called valkyrie-app with the tag v0.0.1


```
source <(gsutil cat gs://cloud-training/gsp318/marking/setup_marking.sh)
```

```
gcloud config list | grep project
```

```
export PROJECT_ID=$YOUR_PROJECT_ID
```

```
gcloud source repos clone valkyrie-app --project=$PROJECT_ID
```

cd valkyrie-app

```
cat > Dockerfile <<EOF
FROM golang:1.10
WORKDIR /go/src/app
COPY source .
RUN go install -v
ENTRYPOINT ["app","-single=true","-port=8080"]
EOF
```

```
docker build -t valkyrie-app:v0.0.1 .
```

Once you have created the Docker image, and before clicking Check my progress, run step1.sh to perform the local check of your work. After you get a successful response from the local marking you can check your progress.

Create a Docker image and store the Dockerfile

```
cd ~/marking
./step1.sh
```

## Task 2: Test the created Docker image
Launch a container using the image valkyrie-app:v0.0.1. You need to map the host’s port 8080 to port 8080 on the container. Add & to the end of the command to cause the container to run in the background.
When your container is running you will see the page by Web Preview.

```
cd valkyrie-app
```

```
docker run -p 8080:8080 --name valkyrie-app valkyrie-app:v0.0.1 &
```

Once you have your container running, and before clicking Check my progress, run step2.sh to perform the local check of your work. After you get a successful response from the local marking you can check your progress.

Test the created Docker image

```
cd ~/marking
./step2.sh
```

## Task 3: Push the Docker image in the Container Repository
Push the Docker image valkyrie-app:v0.0.1 into the Container Registry.
Make sure you re-tag the container to gcr.io/YOUR_PROJECT/valkyrie-app:v0.0.1.
Push the Docker image in the Google Container Repository

```
docker tag valkyrie-app:v0.0.1 gcr.io/$PROJECT_ID/valkyrie-app:v0.0.1
docker images
docker push gcr.io/$PROJECT_ID/valkyrie-app:v0.0.1
```

## Task 4: Create and expose a deployment in Kubernetes
Kurt created the deployment.yaml and service.yaml to deploy your new container image to a Kubernetes cluster (called valkyrie-dev). The two files are in valkyrie-app/k8s.
Remember you need to get the Kubernetes credentials before you deploy the image onto the Kubernetes cluster.
Before you create the deployments make sure you check the deployment.yaml and service.yaml files. Kurt thinks they need some values set (he thinks he left some placeholder values).
You can check the load balancer once it’s available.
Create and expose a deployment in Kubernetes

```
cd valkyrie-app/k8s
```

```
gcloud container clusters get-credentials valkyrie-dev --zone us-east1-d
```

Use a text editor to modify deployment.yaml and replace IMAGE_HERE with gcr.io/YOUR_PROJECT_ID/valkyrie-app:v0.0.1
kubectl create -f <filename>

Use kubectl create -f <filename> command to deploy deployment.yaml and service.yaml

```
kubectl create -f deployment.yaml
kubectl create -f service.yaml
```

## Task 5: Update the deployment with a new version of valkyrie-app
Before deploying the new code, increase the replicas from 1 to 3 to ensure you don't cause an outage.

Increase the replicas from 1 to 3

```
kubectl scale deployment valkyrie-dev --replicas 3
```

Kurt made changes to the source code (he put the changes in a branch called kurt-dev). You need to merge kurt-dev into master (you should use git merge origin/kurt-dev).
Build the new code as version v0.0.2 of valkyrie-app, push the updated image to the Container Repository, and then redeploy to the valkyrie-dev cluster. You will know you have the new v0.0.2 version because the titles for the cards will be green.
```
cd valkyrie-app
```

```
git merge origin/kurt-dev
```

Update the deployment with a new version of valkyrie-app

```
docker build -t valkyrie-app:v0.0.2 .
docker tag valkyrie-app:v0.0.2 gcr.io/$PROJECT_ID/valkyrie-app:v0.0.2
docker images
docker push gcr.io/$PROJECT_ID/valkyrie-app:v0.0.2
```

```
kubectl edit deployment valkyrie-dev
```

```
kubectl set image deployment valkyrie-dev backend=gcr.io/$PROJECT_ID/valkyrie-app:v0.0.2 frontend=gcr.io/$PROJECTID/valkyrie-app:v0.0.2
```

## Task 6: Create a pipeline in Jenkins to deploy your app  
This process of building the container and pushing to the container repository can be automated using Jenkins. There is a Jenkins deployment in your valkyrie-dev cluster - connect to Jenkins and configure a job to build when you push a change to the source code.
Remember with Jenkins:
- Get the password with 
```
printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
```
- Connect to the Jenkins console using the commands below (but make sure you don't have a running container docker ps; if you do, kill it):
	
```
docker ps
docker stop [container id]
/** Optional */	
docker rm [container id]
```
	
```
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
```
- Setup your credentials to use Google Service Account from metadata.
- Create a pipeline job that points to your */master branch on your source code.
Make two changes to your files before you commit and build:
- Edit valkyrie-app/Jenkinsfile and change YOUR_PROJECT to your actual project id.
- Edit valkyrie-app/source/html.go and change the two occurrences of green to orange.
Use git to:
- Add all the changes then commit those changes to the master branch.
- Push the changes back to the repository.
When you are ready, manually trigger a build (the initial build will take some time, so just monitor the process). The build will replace the running containers with containers with different tags; you will see orange colored headings.

Create a pipeline in Jenkins to deploy your app

get your Jenkins admin account password :	
```
printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
```

```
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
```



In the Jenkins user interface, click Credentials in the left navigation.

- Click Jenkins

- Click Global credentials (unrestricted).

- Click Add Credentials in the left navigation.

- Select Google Service Account from metadata from the Kind drop-down and click OK.



Click Jenkins > New Item in the left navigation:

- Name the project valkyrie-dev, then choose the Pipeline option and click OK.

- On the next page, in the Branch Sources section, click Add Source and select git.

- Paste the HTTPS clone URL of your sample-app repo in Cloud Source Repositories https://source.developers.google.com/p/YOUR_PROJECT_ID/r/valkyrie-app into the Project Repository field. Remember to replace YOUR_PROJECT_ID with your GCP Project ID.

- From the Credentials drop-down, select the name of the credentials you created when adding your service account in the previous steps.


In Cloud Shell : 
	
* Open Jenkinsfile file in a text editor, and replace YOUR_PROJECT with your GCP project ID.

* Open source/html.go file in a text editor, and change the color of headings from green to orange.

	
```
git config --global user.email $PROJECT_ID
git config --global user.name $PROJECT_ID
```
```
git add *
git commit -m 'green to orange'
git push origin master
```

Manually trigger a project build in Jenkins
	
	
	
