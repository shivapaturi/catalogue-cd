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
        
        stage('Deploy') {
            steps {
                script{
                    withAWS(region:'us-east-1',credentials:'aws-creds') {
                        sh """
                            aws eks update-kubeconfig --region ${REGION} --name ${PROJECT}-${params.deploy_to}
                            kubectl get nodes
                        """
                    }
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