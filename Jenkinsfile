pipeline {
    agent any
    
    options {
        ansiColor('xterm')
    }
    
    environment {
        registryUrl="demoossacr.azurecr.io"
        registry = "demoossacr.azurecr.io/sampleapp"
        registryCredential = 'ACR'
        aksCredential = 'AKS'
        dockerImage = ''
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/optus']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/arubasu/linux_tweet_app']]])
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }

        stage('Deploy Docker Image to ACR') {
            steps {
                    script {
                            docker.withRegistry( "https://${registryUrl}", registryCredential ) {
                                dockerImage.push()
                                dockerImage.push('latest')
                            }
                    }
            }
        }
        
        stage('Remove Unused local docker image') {
            steps{
               sh "docker rmi $registry:$BUILD_NUMBER"
               sh "docker rmi $registry:latest"
            }
        }
        
        stage('Deploy to k8s') {
            steps{
                /* We have copied the kune_config inside Agent VM -> ~/aksdemo */
                sh '''
                    sed -i "s/:latest/:$BUILD_NUMBER/g" manifests/deployment.yml
                    kubectl apply -f manifests/deployment.yml --kubeconfig ~/aksdemo
                    kubectl apply -f manifests/service.yml --kubeconfig ~/aksdemo
                '''
            }
        }
    }
}
