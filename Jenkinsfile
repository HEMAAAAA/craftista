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
            sh 'docker logout'
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
        withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
            sh 'echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin'
            
            sh """
                docker build -t ${env.DOCKERHUB_REPO}/${serviceName}:${env.VERSION} .
                docker tag ${env.DOCKERHUB_REPO}/${serviceName}:${env.VERSION} ${env.DOCKERHUB_REPO}/${serviceName}:latest
                docker push ${env.DOCKERHUB_REPO}/${serviceName}:${env.VERSION}
                docker push ${env.DOCKERHUB_REPO}/${serviceName}:latest
            """
        }
        
        echo "Successfully pushed ${env.DOCKERHUB_REPO}/${serviceName}:${env.VERSION} to DockerHub"
    }
}