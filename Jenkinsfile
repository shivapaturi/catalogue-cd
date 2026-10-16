pipeline {
    agent  {
        label 'AGENT-1'
    }
    environment { 
        appVersion = ''
        REGION = "us-east-1"
        ACC_ID = "160885265516"
        PROJECT = "roboshop"
        COMPONENT = "catalogue"
    }
    options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }
    parameters {
        string(name: 'appVersion', description: 'Image version of the application')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the Environment')
    }
    // Build
    stages {
        
        stage('Deploy') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${REGION}") {
                    sh """
                        aws eks update-kubeconfig \
                        --region ${REGION} \
                        --name "${PROJECT}-${params.deploy_to}"

                        which kubectl
                        kubectl get nodes

                        kubectl apply -f 01-namespace.yaml
                        kubectl apply -f application.yaml
                    """
                }
            }
        }
        
    }

        

    post{
        always{
            echo 'I will always say Hello again!'
            cleanWs()
        }
        success {
            echo 'I will run if success'
        }
        failure {
            echo 'I will run if failure'
        }
        aborted {
            echo 'pipeline is aborted'
        }
    }
}