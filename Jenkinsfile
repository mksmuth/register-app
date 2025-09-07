pipeline {
    agent { label 'jenkins-agent' }
    tools {
        jdk 'Java17'
        maven 'Maven-3'
    
    }
	 environment {
	    APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "muthukumar001"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
		 JENKINS_TOKEN = credentials("JENKINS_TOKEN")

	 }
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/mksmuth/register-app/'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

       }

       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }
       stage("SonarQube Analysis"){
           steps {
	           script {
		            withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                    sh "mvn sonar:sonar"
                    }
               }
           }
       }
		 stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/',DOCKER_PASS) {
						def
                        docker_image = docker.build ("${IMAGE_NAME}")
        
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
					}
				}
			}
		 }
		stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image muthukumar001/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }
		stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
			   }
		   }
		}
		stage("CD pipeline trigger") {
    steps {
        script {
            echo "Triggering CD pipeline with IMAGE_TAG=${IMAGE_TAG}"
            sh """
                curl -v -k --user clouduser:${JENKINS_TOKEN} \
                     -X POST \
                     -H 'cache-control: no-cache' \
                     -H 'content-type: application/x-www-form-urlencoded' \
                     --data-urlencode "IMAGE_TAG=${IMAGE_TAG}" \
                     'https://ec2-54-226-51-188.compute-1.amazonaws.com:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'
            """
           }
         }
      }
	}
}
		

