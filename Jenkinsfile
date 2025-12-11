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
                
                echo "Check"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE} StudyBud'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${DOCKER_IMAGE}
                    '''
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
            echo "Deployment succeeded! Visit http://${EC2_HOST}:${APP_PORT}"
        }
        failure {
            echo "Deployment failed."
        }
    }
}