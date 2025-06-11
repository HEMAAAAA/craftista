pipeline {
    agent any
    
    environment {
        DOCKERHUB_REPO = "hema995"  
        VERSION = "${BUILD_NUMBER}"
        GIT_CREDENTIALS = credentials('git-credentials')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build and Push Images') {
            parallel {
                stage('Frontend') {
                    steps {
                        dir('frontend') {
                            buildAndPushDockerImage('craftista-frontend')
                        }
                    }
                }
                
                stage('Catalogue') {
                    steps {
                        dir('catalogue') {
                            buildAndPushDockerImage('craftista-catalogue')
                        }
                    }
                }
                
                stage('Recommendation') {
                    steps {
                        dir('recommendation') {
                            buildAndPushDockerImage('craftista-recommendation')
                        }
                    }
                }
                
                stage('Voting') {
                    steps {
                        dir('voting') {
                            buildAndPushDockerImage('craftista-voting')
                        }
                    }
                }
            }
        }
        
        
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo 'All Docker images built and pushed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for details.'
        }
    }
}

def buildAndPushDockerImage(String serviceName) {
    script {
        // Login to DockerHub
        withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
            sh "docker login -u ${env.DOCKERHUB_USERNAME} -p ${env.DOCKERHUB_PASSWORD}"
            // Build the Docker image
            sh "docker build -t ${env.DOCKERHUB_REPO}/${serviceName}:${env.BUILD_NUMBER} ."
        }
        
        // Push the Docker image
        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
            sh "docker push ${env.DOCKERHUB_REPO}/${serviceName}:${env.BUILD_NUMBER}"
            
            // Optionally push as 'latest'
            sh "docker tag ${env.DOCKERHUB_REPO}/${serviceName}:${env.BUILD_NUMBER} ${env.DOCKERHUB_REPO}/${serviceName}:latest"
            sh "docker push ${env.DOCKERHUB_REPO}/${serviceName}:latest"
        }
        
        echo "Successfully pushed ${env.DOCKERHUB_REPO}/${serviceName}:${env.BUILD_NUMBER} to DockerHub"
    }
}
