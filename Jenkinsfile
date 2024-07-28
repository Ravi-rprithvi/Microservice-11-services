pipeline {
    agent any

    stages {
        stage('Deploy To Kubernetes') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'education-eks-xzdDGnz8', contextName: '', credentialsId: 'kubecred', namespace: 'webapps', serverUrl: 'https://3D8922FA6171E0CCFFAB5572C8A1C6E9.gr7.us-east-1.eks.amazonaws.com']]) {
                    sh "kubectl apply -f deployment-service.yml"
                    
                }
            }
        }
        
        stage('verify Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'education-eks-xzdDGnz8', contextName: '', credentialsId: 'kubecred', namespace: 'webapps', serverUrl: 'https://3D8922FA6171E0CCFFAB5572C8A1C6E9.gr7.us-east-1.eks.amazonaws.com']]) {
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }
}

