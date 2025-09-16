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
		    steps {
		        sh '''
		            docker run --rm \
		                -v /var/run/docker.sock:/var/run/docker.sock \
		                -v $(pwd):/workspace \
		                -w /workspace \
		                aquasec/trivy:latest \
		                image --severity HIGH,CRITICAL --no-progress --format table --output trivy-scan-report.txt ${DOCKER_HUB_REPO}:latest
		        '''
		        
		        archiveArtifacts artifacts: 'trivy-scan-report.txt', fingerprint: true
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
