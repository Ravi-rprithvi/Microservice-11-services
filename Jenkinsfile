pipeline {
    agent any

    stages {
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockercred', toolName: 'docker') {
                        sh "docker build -t nishantg98/emailservice:latest ."
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockercred', toolName: 'docker') {
                        sh "docker push nishantg98/emailservice:latest "
                    }
                }
            }
        }
    }
}
