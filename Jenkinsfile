pipeline {
    agent {
        node {
            label 'AGENT-1'
        }
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
        string(name: 'appVersion', description: 'Image version of the application')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the Environment')
    }

    stages {
         {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    env.APP_VERSION = packageJson.version

                    echo "Package version: ${env.APP_VERSION}"
                }
            }
        }

    }   
    stages {
            stage('Deploy') {
                steps {}
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
                                #helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n $PROJECT .
                                kubectl apply -f application.yaml
                            """
                        }
                    }
                }
            }
    post {
        always {
            echo 'I will always say Hello again!'
            deleteDir()
        }

        success {
            echo 'Hello success!'
        }

        failure {
            echo 'Hello failure!'
        }
    }
}