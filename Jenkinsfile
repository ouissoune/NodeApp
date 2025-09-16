def dockerImage
pipeline {
	agent none
	environment {
		DOCKER_HUB_CREDENTIALS_ID = 'jen-dockerhub'
		DOCKER_HUB_REPO = 'kilmerx/nodeapp'
	}
	stages {	
		stage('Install node dependencies'){
				agent {
		docker {
			image 'node:16-alpine'
		}
	}
			steps {
				sh 'npm install'
			}
		}
		stage('Test Code'){
				agent {
		docker {
			image 'node:16-alpine'
		}
	}
			steps {
				sh 'npm test'
			}
		}
		stage('Build Docker Image'){
			steps {
				script {
					dockerImage = docker.build("${DOCKER_HUB_REPO}:latest")
				}
			}
		}
		stage('Trivy Scan') {
		    agent {
		        docker {
		            image 'bitnami/trivy:latest'
		            args '--entrypoint=""'
		        }
		    }
		    steps {
		        sh 'trivy --severity HIGH,CRITICAL --no-progress image --format table -o trivy-scan-report.txt ${DOCKER_HUB_REPO}:latest'
		    }
		}

		stage('Push Image to DockerHub'){
			steps {
				script {
					docker.withRegistry('https://registry.hub.docker.com', "${DOCKER_HUB_CREDENTIALS_ID}"){
						dockerImage.push('latest')
					}
				}
			}
		}
}
}
