pipeline {
    agent any

    tools {
        nodejs 'node'
    }

    environment {
        // Branch-based settings
        APP_NAME     = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        APP_PORT     = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
        DOCKER_IMAGE = "aigulsadykova/${APP_NAME}:v1.0"
    }

    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        def app = docker.build("${DOCKER_IMAGE}")
                        app.push()
                    }
                }
            }
        }

        stage('Scan with Trivy') {
            steps {
                sh "trivy image --exit-code 0 --severity HIGH,CRITICAL ${DOCKER_IMAGE}"
                sh "trivy image --exit-code 1 --severity CRITICAL ${DOCKER_IMAGE}"
            }
        }

        stage('Trigger Deploy Pipeline') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        build job: 'Deploy_to_main'
                    } else if (env.BRANCH_NAME == 'dev') {
                        build job: 'Deploy_to_dev'
                    }
                }
            }
        }
    }
}

      echo "Pipeline finished for ${env.BRANCH_NAME}"
    }
  }
}
