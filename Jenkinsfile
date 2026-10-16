pipeline {
    agent  {
        label 'AGENT-1'
    }
    environment { 
        appVersion = ''
        REGION = "us-east-1"
        ACC_ID = "211125615103"
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
        stage('Check Status') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: "${REGION}") {
                        sh """
                            export PATH=/usr/local/bin:\$PATH

                            kubectl rollout status \
                            deployment/${COMPONENT} \
                            --timeout=120s \
                            -n ${PROJECT}
                        """
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: "${REGION}") {
                        sh """
                            export PATH=/usr/local/bin:\$PATH

                            aws eks update-kubeconfig \
                            --region ${REGION} \
                            --name "${PROJECT}-${params.deploy_to}"

                            kubectl get nodes

                            kubectl apply -f 01-namespace.yaml

                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" \
                            values-${params.deploy_to}.yaml

                            kubectl apply -f application.yaml
                        """
                    }
                }
            }
        }

        
        // API Testing
        stage('Functional Testing'){
            when{
                expression { params.deploy_to = "dev" }
            }
             steps{
                script{
                    echo "Run functional test cases"
                }
            }
        }
        // All components testing
        stage('Integration Testing'){
            when{
                expression { params.deploy_to = "qa" }
            }
             steps{
                script{
                    echo "Run Integration test cases"
                }
            }
        }
        stage('PROD Deploy') {
            when{
                expression { params.deploy_to = "prod" }
            }
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        sh """
                            echo "get cr number"
                            echo "check with in the deployment window"
                            echo "is CR approved"
                            echo "trigger PROD deploy"
                        """
                    }
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
            echo 'Hello Success'
        }
        failure { 
            echo 'Hello Failure'
        }
    }
}