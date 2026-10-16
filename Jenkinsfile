pipeline {
    agent { label 'AGENT-1' }

    environment {
        REGION    = "us-east-1"
        ACC_ID    = "160885265516"
        PROJECT   = "roboshop"
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

    stages {
        stage('Validate') {
            steps {
                script {
                    if (!params.appVersion?.trim()) {
                        error "appVersion parameter is required"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: "${REGION}") {
                        sh """
                            aws eks update-kubeconfig --region ${REGION} --name ${PROJECT}-${params.deploy_to}
                            kubectl get nodes
                            kubectl apply -f 01-namespace.yaml
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                            helm upgrade --install ${COMPONENT} -f values-${params.deploy_to}.yaml -n ${PROJECT} .
                        """
                    }
                }
            }
        }

        stage('Check Status') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: "${REGION}") {
                        sh "aws eks update-kubeconfig --region ${REGION} --name ${PROJECT}-${params.deploy_to}"

                        def deploymentStatus = sh(returnStdout: true,
                            script: "kubectl rollout status deployment/${COMPONENT} --timeout=60s -n ${PROJECT} || echo FAILED").trim()

                        if (deploymentStatus.contains("successfully rolled out")) {
                            echo "Deployment is success"
                        } else {
                            sh """
                                helm rollback ${COMPONENT} -n ${PROJECT}
                                sleep 20
                            """
                            def rollbackStatus = sh(returnStdout: true,
                                script: "kubectl rollout status deployment/${COMPONENT} --timeout=60s -n ${PROJECT} || echo FAILED").trim()

                            if (rollbackStatus.contains("successfully rolled out")) {
                                error "Deployment is Failure, Rollback Success"
                            } else {
                                error "Deployment is Failure, Rollback Failure. Application is not running"
                            }
                        }
                    }
                }
            }
        }

        stage('Functional Testing') {
            when { expression { params.deploy_to == "dev" } }
            steps { echo "Run functional test cases" }
        }

        stage('Integration Testing') {
            when { expression { params.deploy_to == "qa" } }
            steps { echo "Run Integration test cases" }
        }

        stage('PROD Deploy') {
            when { expression { params.deploy_to == "prod" } }
            steps {
                sh """
                    echo "get cr number"
                    echo "check with in the deployment window"
                    echo "is CR approved"
                    echo "trigger PROD deploy"
                """
            }
        }
    }

    post {
        always  { echo 'I will always say Hello again!' }
        success { echo 'Hello Success' }
        failure { echo 'Hello Failure' }
        cleanup { deleteDir() }
    }
}