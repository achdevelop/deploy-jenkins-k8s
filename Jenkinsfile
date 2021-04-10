pipeline {
    agent { 
        kubernetes{
            label 'jenkins-slave'
        }
        
    }
    environment{
        DOCKER_USERNAME = credentials('DOCKER_USERNAME')
        DOCKER_PASSWORD = credentials('DOCKER_PASSWORD')
    }
    stages {
        stage('docker login') {
            steps{
                sh(script: """
                    docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                """, returnStdout: true) 
            }
        }

        stage('git clone') {
            steps{
                sh(script: """
                    git clone https://github.com:achgeek/deploy-jenkins-k8s.git
                """, returnStdout: true) 
            }
        }

        stage('docker build') {
            steps{
                sh script: '''
                #!/bin/bash
                cd $WORKSPACE/python
                docker build . --network host -t achdevops/jenkins-python:${BUILD_NUMBER}
                '''
            }
        }

        stage('docker push') {
            steps{
                sh(script: """
                    docker push achdevops/jenkins-python:${BUILD_NUMBER}
                """)
            }
        }

        stage('deploy') {
            steps{
                sh script: '''
                #!/bin/bash
                cd $WORKSPACE/
                #get kubectl for this demo
                curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
                chmod +x ./kubectl
                ./kubectl apply -f ./manifest/configmap.yaml
                ./kubectl apply -f ./manifest/secret.yaml
                cat ./manifest/deployment.yaml | sed s/1.0.0/${BUILD_NUMBER}/g | ./kubectl apply -f -
                ./kubectl apply -f ./manifest/service.yaml
                '''
        }
    }
}
}