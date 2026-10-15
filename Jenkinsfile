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
            defaultValue: 'v1',
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
                        error('appVersion cannot be empty')
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
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        sh """
                            aws eks update-kubeconfig --region ${REGION} --name "${PROJECT}-${params.deploy_to}"
                            kubectl get nodes
                            kubectl apply -f 01-namespace.yaml
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                            #helm upgrade --install ${COMPONENT} -f values-${params.deploy_to}.yaml -n ${PROJECT} .
                            kubectl apply -f application.yaml
                        """
                    }
                }
            }
        }

        stage('Check Status') {
            steps {
                script {
                    withAWS(
                        credentials: 'aws-creds',
                        region: "${REGION}"
                    ) {
                        def deploymentStatus = sh(
                            returnStdout: true,
                            script: """
                                kubectl rollout status \
                                  deployment/${COMPONENT} \
                                  -n ${PROJECT} \
                                  --timeout=120s 2>&1 || echo FAILED
                            """
                        ).trim()

                        if (deploymentStatus.contains(
                            'successfully rolled out'
                        )) {
                            echo 'Deployment is successful'
                        } else {
                            echo "Deployment failed: ${deploymentStatus}"
                            echo 'Performing Helm rollback...'

                            sh """
                                helm rollback "${COMPONENT}" -n "${PROJECT}"
                            """

                            def rollbackStatus = sh(
                                returnStdout: true,
                                script: """
                                    kubectl rollout status \
                                      deployment/${COMPONENT} \
                                      -n ${PROJECT} \
                                      --timeout=120s 2>&1 || echo FAILED
                                """
                            ).trim()

                            if (rollbackStatus.contains(
                                'successfully rolled out'
                            )) {
                                error(
                                    'New deployment failed; rollback succeeded. ' +
                                    'Jenkins build marked as FAILURE.'
                                )
                            } else {
                                error(
                                    'Deployment and rollback verification failed. ' +
                                    "Status: ${rollbackStatus}"
                                )
                            }
                        }
                    }
                }
            }
        }
        // API Testing
        stage('Functional Testing') {
            when{
                expression { params.deploy_to = "dev" }
            }
            steps {
                script {
                    echo "Run fuctional test cases"
                }
            }
        }
        // All Components Testing
        stage('Integrational Testing') {
            when{
                expression { params.deploy_to = "qa" }
            }
            steps {
                script {
                    echo "Run fuctional test cases"
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