pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'your-dockerhub-username/devops-app'
    }

    stages {
        stage('Build') {
            steps {
                sh 'docker-compose build'
            }
        }

        stage('Test') {
            steps {
                sh 'docker-compose up -d'
                sh 'docker compose exec -T app curl -f http://localhost:3000 || exit 1'
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo "$PASS" | docker login -u "$USER" --password-stdin
                    docker tag kelpassignment-app $DOCKER_IMAGE
                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to AWS') {
            steps {
                // Use ECS CLI, Helm for EKS, or SSH commands to deploy to EC2
                echo 'Deploy step here...'
            }
        }
    }
}
