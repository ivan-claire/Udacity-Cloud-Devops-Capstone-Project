# Udacity-Cloud-Devops-Capstone-Project
 CI/CD pipeline for micro services applications with either blue/green deployment using EKS
 
## Summary
A Blue/Green deployment is a way of accomplishing a zero-downtime upgrade to an existing application.
The “Blue” version will be deployed first and the “Green” version which is the newer version of the application will then deployed.
A rollout status of the newer version of the deployment(green) is done to check if it is running and ready (Health Check).
If it is ready, the load balancer service is updated to point to the new deployment and the blue version is deleted. 
The user therefore experiences a seamless switch between the blue and green verisons of the application.
The Blue/Green deployment strategy is therefore terminated. 
All of this is done only after the docker image has been created from the Dockerfile. 
The dockerfile is used to deploy an html page (myapp).

## Setting Up Environment on EC2 Ubuntu Instance
- Install Jenkins 
- Install aws-cli and eksctl for cloudformation commands on eks. Use this guide as reference
https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html

## Creating K8s Cluster
Firstly, a K8s cluster is created in EKS using Cloud formation and the eksctl commmand line tool.

## Build and Pull Docker Image to the Docker Hub
The docker image is built from the dockerfile and pushed to the hub.
the image is tagged with a hash of the git commit inorder to distinguish the commits.

## Blue/Green Deployment Strategy is Performed
The first version of the app: myapp-1.00 is deployed using kubectl and the blue-deployment.yml file
and the load balancer service defined in the service.yml file points to it.
Then the green version: myapp-1.01 is deployed and a health check done to ensure it runs correctly.
Once it's confirmed that the depmoyment is healthy,
the loadbalancer service is switched to the new version of the app.
![Pipeline Image](https://github.com/ivan-claire/Udacity-Cloud-Devops-Capstone-Project/blob/master/performing healthcheck.png)



