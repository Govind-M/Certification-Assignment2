pipeline {
    agent {label 'Kubernetes Agent'}
    
    triggers {
        githubPush()  // Automatically triggers when a push is made to GitHub
    }
    
    environment {
	DOCKER_USERNAME = 'manojgovind'
        IMAGE_NAME = "my-apache-website-image"
	IMAGE_TAG = 'latest'
	IMAGE = "${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
        CONTAINER_NAME = "my-apache-website-container"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch:'gh-pages', url:'https://github.com/Govind-M/Certification-Assignment2.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE -f dockerfile.txt .'
            }
        }
        stage('Stop and Remove Existing Container') {
            steps {
                sh 'docker stop $CONTAINER_NAME || true'
                sh 'docker rm $CONTAINER_NAME || true'
            }
        }
	stage('Push to DockerHub') {
      		steps {
        		withCredentials([string(credentialsId: 'dockerhub-pass', variable: 'DOCKER_PASS')]) {
          		sh '''
            		echo $DOCKER_PASS | docker login -u $DOCKER_USERNAME --password-stdin
            		docker push $IMAGE
          		'''
       			 	}
     		 	   }
			}
	 stage('Deploy to Kubernetes') {
      		steps {
        		sh '''
          		kubectl apply -f deployment.yaml
          		kubectl apply -f service.yaml
        		'''
      			}
    		}
        }
}
