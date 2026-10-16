pipeline {

    agent {
        label 'AGENT-1'
    }

    environment {
        REGION    = 'us-east-1'
        ACC_ID    = '211125615103'
        PROJECT   = 'roboshop'
        COMPONENT = 'catalogue'
    }

    stages {

        stage('Deploy') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${REGION}") {
                    sh """
                        export PATH=/usr/local/bin:\$PATH

                        aws eks update-kubeconfig \
                          --region ${REGION} \
                          --name "${PROJECT}-${params.deploy_to}"

                        kubectl get nodes

                        kubectl apply -f 01-namespace.yaml
                        kubectl apply -f application.yaml
                    """
                }
            }
        }

        stage('Check Status') {
            steps {
                sh """
                    export PATH=/usr/local/bin:\$PATH
                    kubectl rollout status \
                      deployment/${COMPONENT} \
                      -n ${PROJECT} \
                      --timeout=120s
                """
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