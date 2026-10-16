pipeline {
    agent {
        node { label 'AGENT-1' }
    }

    environment {
        REGION    = "us-east-1"
        ACC_ID    = "211125615103"
        PROJECT   = "roboshop"
        COMPONENT = "catalogue"
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    parameters {
        string(name: 'appVersion', description: 'Which app version you want to deploy')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the environment')
    }

    stages {
        stage('Validate') {
            steps {
                script {
                    if (!params.appVersion?.trim()) {
                        error "appVersion parameter is required"
                    }
                    echo "Deploying ${COMPONENT}:${params.appVersion} to ${params.deploy_to}"
                }
            }
        }

        stage('Deploy') {
            steps {
                withAWS(region: "${REGION}", credentials: 'aws-creds') {
                    sh """
                        aws eks update-kubeconfig --region ${REGION} --name ${PROJECT}-${params.deploy_to}
                        kubectl get nodes
                    """
                }
            }
        }
    }

    post {
        always  { echo 'I will always say Hello again!' }
        success { echo 'I will run if success' }
        failure { echo 'I will run if failure' }
        aborted { echo 'pipeline is aborted' }
        cleanup { cleanWs() }
    }
}