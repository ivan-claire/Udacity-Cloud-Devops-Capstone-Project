pipeline {

	agent any
	stages {

		stage('Create kubernetes cluster') {
				steps {
					withAWS(region:'us-east-1', credentials:'aws-static') {
						sh '''
							# eksctl create cluster \
							# --name udacity-capstone \
							# --version 1.13 \
							# --nodegroup-name standard-workers \
							# --node-type t2.small \
							# --nodes 2 \
							# --nodes-min 1 \
							# --nodes-max 3 \
							# --node-ami auto \
							# --region us-east-1 \
							# --zones us-east-1a \
							# --zones us-east-1b \
							# --zones us-east-1c 

				#configuring kubectl
				aws eks --region us-east-1 update-kubeconfig --name udacity-capstone
						'''
					}
				}
		}

		stage('Lint HTML') {
				steps {
					sh 'tidy -q -e *.html'
				}
			}

		stage('Hashing images') {
		    steps {
			script {
			    env.GIT_HASH = sh(
				script: "git show --oneline | head -1 | cut -d' ' -f1",
				returnStdout: true
			    ).trim()
			}
		    }
		}

		stage('Build & Push Docker Image To Dockerhub') {
			steps {
				 withDockerRegistry([ credentialsId: "dockerhub", url: "" ]) {
					 script {
					    dockerImage = docker.build("awony/capstone:${env.GIT_HASH}")
					    dockerImage.push()
					 }
				 }
			}
               }
		
		stage('Perform a Blue Green Deployment') {
			steps {
				withAWS(region:'us-east-1', credentials:'aws-static') {
					sh '''
					kubectl config use-context arn:aws:eks:us-east-1:023700642655:cluster/udacity-capstone
					kubectl delete deployment blue
					kubectl apply -f ./blue-deployment.yml
					kubectl apply -f ./service.yml
					export BlueVersion=$(kubectl get service bluegreenlb -o=jsonpath='{.spec.selector.version}') #find deployed version
					kubectl get deployment myapp-$BlueVersion -o=yaml | sed -e "s/$BlueVersion/$GIT_HASH/g" | kubectl apply  -f ./green-deployment.yml #Deploy new version
					'''
				}
			}
		}

		stage('Perform Health Check and Update Service to Green Deployment') {
			steps {
				withAWS(region:'us-east-1', credentials:'aws-static') {
					sh '''
					kubectl get pods
					kubectl get deployments
					kubectl rollout status deployment/myapp-$GIT_HASH #Health Check
					kubectl apply -f ./service.yml  #Update Service YAML with Green version
					kubectl delete deployment myapp-$BlueVersion #Delete blue version
					'''
				}
			}
        }

	}
}
