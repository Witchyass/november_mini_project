pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'susan22283/WTF_November_mini_project:latest'
        EC2_USER     = 'ubuntu'
        EC2_HOST     = '52.91.168.143'
        APP_PORT     = '8000'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Witchyass/november_mini_project.git',
                        credentialsId: 'github-token'
                    ]]
                ])
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build from StudyBud subdirectory (where manage.py lives)
                    docker.build("${DOCKER_IMAGE}", "StudyBud")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-creds') {
                        docker.image("${DOCKER_IMAGE}").push()
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-creds', keyFileVariable: 'EC2_KEY')]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no -i \$EC2_KEY ${EC2_USER}@${EC2_HOST} '
                            if ! command -v docker &> /dev/null; then
                                sudo apt-get update
                                sudo apt-get install -y docker.io
                                sudo usermod -aG docker ${EC2_USER}
                                newgrp docker 2>/dev/null || true
                            fi
                            sudo docker pull ${DOCKER_IMAGE}
                            sudo docker rm -f python_app 2>/dev/null || true
                            sudo docker run -d --name python_app -p ${APP_PORT}:${APP_PORT} ${DOCKER_IMAGE}
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "deployment succeeded! Visit http://${EC2_HOST}:${APP_PORT}"
        }
        failure {
            echo "deployment failed."
        }
    }
}