pipeline {
    agent any

    stages {
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockercred', toolName: 'docker') {
                        sh "docker build -t 8770406568/currencyservice:latest ."
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockercred', toolName: 'docker') {
                        sh "docker push 8770406568/emailservice:latest "
                    }
                }
            }
        }
    }
}
