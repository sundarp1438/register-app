pipeline {
    agent { label 'rocky-linux-09' }
    tools {
        jdk 'JAVA_HOME'
        maven 'MAVEN_HOME'
    }
    environment {
	    APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "sundarp1985"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	    JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/sundarp1438/register-app'
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

       stage("Quality Gate") {
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }	
            }

        }
	stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
	  stage('Build Docker Image') {
            steps {
                script{
                    sh 'docker build -t sundarp1985/register-app-pipeline:latest .'
                }
            }
         }
	stage('Containerize And Test') {
            steps {
                script{
                    sh 'docker run -d --name register-app sundarp1985/register-app-pipeline:latest && sleep 10 && docker stop register-app'
                }
            }
        }
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }

       }
       stage("TRIVY Image Scan"){
            steps{
                sh "trivy image avian19/netflix:latest > trivyimage.txt" 
            }
        }

       stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi -f ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi -f ${IMAGE_NAME}:latest"
               }
          }
       }

       //stage("Trigger CD Pipeline") {
       //     steps {
       //         script {
       //             sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-13-233-42-35.ap-south-1.compute.amazonaws.com:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'"
       //         }
      //      }
    //   }
    }

    // post {
    //    failure {
    //          emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
    //                   subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed", 
    //                   mimeType: 'text/html',to: "ashfaque.s510@gmail.com"
    //   }
    //   success {
    //         emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
    //                  subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful", 
    //                  mimeType: 'text/html',to: "ashfaque.s510@gmail.com"
    //   }      
   // }
}
