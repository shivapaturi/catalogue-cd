pipeline {
    agent { label 'AGENT-1' }

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
        booleanParam(name: 'deploy', defaultValue: true, description: 'Trigger CD after build?')
    }

    stages {
        stage('Read Version') {
            steps {
                script {
                    // catalogue is a Node.js app, so the version lives in package.json
                    def packageJson = readJSON file: 'package.json'
                    env.appVersion = packageJson.version
                    echo "App version: ${env.appVersion}"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: "${REGION}") {
                        sh """
                            aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com
                            docker build -t ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}:${env.appVersion} .
                            docker push ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}:${env.appVersion}
                        """
                    }
                }
            }
        }

        stage('Trigger CD') {
            when { expression { params.deploy } }
            steps {
                build job: '/ROBOSHOP/catalogue-cd',   // full path to the CD job
                      parameters: [
                          string(name: 'appVersion', value: "${env.appVersion}"),
                          string(name: 'deploy_to',  value: 'dev')
                      ],
                      propagate: false,   // CI stays green even if CD fails
                      wait: false         // don't block the CI agent
            }
        }
    }

    post {
        always  { echo 'CI finished' }
        cleanup { deleteDir() }
    }
}