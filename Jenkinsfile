pipeline {
    agent any

    tools {
        nodejs 'nodejs-20'   // the name you configured in Global Tools
    }

    environment {
        // Branch-based settings
        APP_NAME = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        APP_PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
        DOCKER_IMAGE = "${APP_NAME}:v1.0"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", 
                    url: 'git@github.com:apohalo/cicd-jenkins.git', 
                    credentialsId: 'github-ssh-creds'
            }
        }

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

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Stop and remove previous containers with same name
                    sh """
                        docker ps -q --filter name=${APP_NAME} | xargs -r docker stop
                        docker ps -aq --filter name=${APP_NAME} | xargs -r docker rm
                    """
                    // Run new container
                    sh "docker run -d --name ${APP_NAME} --expose ${APP_PORT} -p ${APP_PORT}:3000 ${DOCKER_IMAGE}"
                }
            }
        }
    }
}
