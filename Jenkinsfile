pipeline {
    agent any 

    tools {nodejs "Nodejs"}

    environment {
        registryCredential = 'dockerhub'
        imageName = 'snehamore213/internalapp'
        dockerImage = ''
        }
    stages {
        stage('Run the tests') {
             agent {
                docker { 
                    image 'node:14-alpine'
                    args '-e HOME=/tmp -e NPM_CONFIG_PREFIX=/tmp/.npm'
                    reuseNode true
                }
            }
            steps {
                echo 'Retrieve source from github. run npm install and npm test' 
                git branch: 'main',
                    url: 'https://github.com/snehamore213/devops_internaldata.git'
                echo 'repo files'
                sh 'ls -a'
                echo 'install dependencies'
                sh 'npm install'
                echo 'Run tests'
                sh 'npm test'
                echo 'Testing completed'
            }
        }
        stage('SonarQube analysis') {
      steps {
        script {
          def scannerHome = tool 'sonarqube';
          withSonarQubeEnv('sonarqube') {
            sh "${scannerHome}/bin/sonar-scanner \ 
            -Dsonar.projectKey=capstone \
            -Dsonar.sources=. \
            -Dsonar.host.url=http://35.188.217.149:9000 \
            -Dsonar.login=9d03446ac6dc5e513d121deb6f3e1360bfb255f9"
          }
         }
        }
       }
       stage("Quality gate") {
      steps {
        script {
          def qualitygate = waitForQualityGate()
          sleep(10)
          if (qualitygate.status != "OK") {
            waitForQualityGate abortPipeline: true
           }
          }
         }
        }
        stage('Building image') {
            steps{
                script {
                    echo 'build the image'
                    dockerImage = docker.build("${env.imageName}:${env.BUILD_ID}")
                    echo "${env.imageName}:${env.BUILD_ID}"
                    echo 'image built'
                }
            }
            }
        stage('Push Image') {
            steps{
                script {
                    echo 'push the image to docker hub'
                    docker.withRegistry('',registryCredential){
                        dockerImage.push("${env.BUILD_ID}")
                  }
                }
            }
        }     
         stage('deploy to k8s') {
             agent {
                docker { 
                    image 'google/cloud-sdk:latest'
                    args '-e HOME=/tmp'
                    reuseNode true
                        }
                    }
            steps {
                echo 'Get cluster credentials'
                sh 'gcloud container clusters get-credentials devops-capstone-cluster --zone us-central1-c'
                sh "kubectl set image deployment/internal-ui internal-uic=${env.imageName}:${env.BUILD_ID}"  
              }
            }     
        stage('Remove local docker images') {
            steps{
                script {
                    echo 'push the image to docker hub' 
                }
                // sh "docker rmi $imageName:latest"
                sh "docker rmi $imageName:$BUILD_NUMBER"
            }
        }
    }
}
