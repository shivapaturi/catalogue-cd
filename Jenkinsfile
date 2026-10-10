
pipeline {
    agent {
        node {
            label 'AGENT-1'
        }
    }

    environment {
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
        string(
            name: 'appVersion',
            defaultValue: '1.0.0',
            description: 'Image version of the application'
        )
        choice(
            name: 'deploy_to',
            choices: ['dev', 'qa', 'prod'],
            description: 'Pick the Environment'
        )
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
                            set -e

                            aws eks update-kubeconfig \
                              --region "${REGION}" \
                              --name "${PROJECT}-${params.deploy_to}"

                            kubectl get nodes

                            kubectl apply -f 01-namespace.yaml

                            sed -i \
                              "s/IMAGE_VERSION/${params.appVersion}/g" \
                              "values-${params.deploy_to}.yaml"

                            kubectl apply -f application.yaml
                        """
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
