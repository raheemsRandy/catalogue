pipeline {

    agent {
        label 'Agent1'
    }

    environment {
        appVersion = ''
        REGION = 'us-east-1'
        ACC_ID ='989088456804'
        PROJECT = 'roboshop'
        COMPONENT = 'catalogue'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    parameters{
        booleanParam( name: 'deploy' , defaultValue:false, description: 'Toggle this value')
    }
    stages {

        stage('Read package.json') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "Package version: ${appVersion}"
                }
            }
        }

        stage('Install dependencies') {
            steps {
                sh 'npm install'
            }
        }
         stage('unit testing') {
            steps {
               echo "unit test"
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: "${REGION}") {

                        sh """
                        aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com
                        
                        docker build -t ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                        
                        docker push ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                        """

                    }
                }
            }
        }
        stage('Trigger deploy') {
            when {
                expression { params.deploy }
            }
            stage('Triggering SG') {
            steps {
                script {
                build job: 'catalogue-cd', 
                 parameters: [
                      string(name: 'appVersion', value: "${appVersion}"),
                      choice(name: 'deploy_to', value: 'dev')
                  ]
                  propagate: false, //up stream job will not fail even sg fails
                  wait: false // vpc will not wait
                }
            }
        }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application'
            }
        }
    }

    post {
        always {
            echo 'Cleaning workspace'
            deleteDir()
        }

        success {
            echo 'Pipeline Success'
        }

        failure {
            echo 'Pipeline Failed'
        }
    }
}