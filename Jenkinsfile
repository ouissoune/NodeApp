def dockerImage
pipeline {
	agent any
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
		            image 'aquasec/trivy:latest'
		            args '-v /var/run/docker.sock:/var/run/docker.sock --entrypoint=""'
					
		        }
		    }
		    steps {
		        sh '''
		            trivy image \
		                --severity HIGH,CRITICAL \
		                --no-progress \
		                --format table \
		                -o trivy-scan-report.txt \
		                ${DOCKER_HUB_REPO}:latest
		        '''
		
		        // Archive the scan report (relative path, not absolute)
		        archiveArtifacts artifacts: 'trivy-scan-report.txt', fingerprint: true
		
		        // Print results in console
		        sh 'cat trivy-scan-report.txt'
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
