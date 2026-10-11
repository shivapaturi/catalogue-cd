
pipeline {
    agent {
        node {
            label 'AGENT-1'
        }
    }

    environment {
        appVersion = ''
        REGION    = 'us-east-1'
        ACC_ID    = '211125615103'
        PROJECT   = 'roboshop'
        COMPONENT = 'catalogue'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    parameters {
        string(name: 'appVersion', description: 'Image version of the application')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the Environment')
    }
    
    stages {
        stage('Validate Parameters') {
            steps {
                script {
                    if (!params.appVersion?.trim()) {
                        error 'appVersion cannot be empty'
                    }

                    echo "Application: ${env.COMPONENT}"
                    echo "Image version: ${params.appVersion}"
                    echo "Environment: ${params.deploy_to}"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    withAWS(
                        credentials: 'aws-creds',
                        region: "${REGION}"
                    ) {
                        sh """
                            aws eks update-kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}"
                            kubectl get nodes
                            kubectl apply -f 01-namespace.yaml
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                            helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n $PROJECT .
                        """
                    }
                }
            }
        }
    
    stage('Check Status') {
        steps {
            script {
                def deploymentStatus = sh(
                    returnStdout: true,
                    script: 'kubectl rollout status deployment/catalogue --request-timeout=30s || echo FAILED'
                ).trim()

                if (deploymentStatus.contains('successfully rolled out')) {
                    echo "Deployment is successful"
                } else {
                    echo "Deployment failed. Performing Helm rollback..."

                    sh """
                        helm rollback ${COMPONENT} -n ${PROJECT}
                        sleep 20
                    """

                    def rollbackStatus = sh(
                        returnStdout: true,
                        script: 'kubectl rollout status deployment/catalogue --request-timeout=30s || echo FAILED'
                    ).trim()

                    if (rollbackStatus.contains('successfully rolled out')) {
                        error("Deployment failed, but rollback succeeded!")
                    } else {
                        error("Deployment failed and rollback failed. Application may not be running!")
                    }
                }
            }
        }
    }

        
    }

    post {
        always {
            echo 'Deployment pipeline finished.'
            deleteDir()
        }

        success {
            echo 'Deployment succeeded!'
        }

        failure {
            echo 'Deployment failed!'
        }
    }
}
