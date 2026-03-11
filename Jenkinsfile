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
        IMAGE = "${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}"
        
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    parameters{
        booleanParam(name: 'deploy', defaultValue: false, description: 'Trigger deployment')
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

        stage('Unit testing') {
            steps {
                echo "Running unit tests"
            }
        }
        // stage('Sonar scan') {
        //     environment {
        //         scannerHome = tool 'sonar-7.2'
        //     }
        //     steps {
        //         withSonarQubeEnv('sonar-7.2') {
        //             sh """
        //                 ${scannerHome}/bin/sonar-scanner
        //             """
        //         }
        //     }
        // }
        //Enable webhook in sonqarqube server and wait for quality gate
        // stage('Quality Gate Check') {
        //     steps {
        //         timeout(time: 1, unit: 'HOURS') { 
                
                    
        //             waitForQualityGate abortPipeline: true
        //         }
        //     }
        // }

         stage('Check Dependabot Alerts') {
            environment { 
                GITHUB_TOKEN = credentials('github-token')
            }
            steps {
                script {
                    // Fetch alerts from GitHub
                    def response = sh(
                        script: """
                            curl -s -H "Accept: application/vnd.github+json" \
                                 -H "Authorization: token ${GITHUB_TOKEN}" \
                                 https://api.github.com/repos/raheemsRandy/catalogue/dependabot/alerts
                        """,
                        returnStdout: true
                    ).trim()

                    // Parse JSON
                    def json = readJSON text: response

                    // Filter alerts by severity
                    def criticalOrHigh = json.findAll { alert ->
                        def severity = alert?.security_advisory?.severity?.toLowerCase()
                        def state = alert?.state?.toLowerCase()
                        return (state == "open" && (severity == "critical" || severity == "high"))
                    }

                    if (criticalOrHigh.size() > 0) {
                        error "❌ Found ${criticalOrHigh.size()} HIGH/CRITICAL Dependabot alerts. Failing pipeline!"
                    } else {
                        echo "✅ No HIGH/CRITICAL Dependabot alerts found."
                    }
                }
            }
        }

                   
        // stage('Docker Build & Push') {
        //     steps {
        //         script {
        //             withAWS(credentials: 'aws-creds', region: REGION) {

        //                 sh """
        //                 aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com
                        
        //                 docker build -t ${IMAGE}:${appVersion} .
                        
        //                 docker push ${IMAGE}:${appVersion}
        //                 """
        //             }
        //         }
        //     }
        // }

        stage('Docker Build') {
                steps {
                    script {
                        withAWS(credentials: 'aws-creds', region: 'us-east-1') {

                            sh """
                                aws ecr get-login-password --region ${REGION} \
                                    | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com

                                # Ensure NO multi-arch builder exists
                                docker buildx rm mybuilder || true

                                # Disable BuildKit to avoid manifest creation
                                export DOCKER_BUILDKIT=0

                                # Build a SINGLE amd64 image
                                docker build --platform=linux/amd64 \
                                    -t ${IMAGE}:${appVersion} .

                                # Push the single image (NO image index)
                                docker push ${IMAGE}:${appVersion}
                            """
                        }
                    }
                }
            }

        stage('Trigger deploy') {
            when {
                expression { params.deploy }
            }

            steps {
                script {
                    build job: 'catalogue-cd',
                        parameters: [
                            string(name: 'appVersion', value: appVersion),
                            string(name: 'deploy_to', value: 'dev')
                        ],
                        propagate: false,
                        wait: false
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deployment stage completed'
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