pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'technicalhrushi/devops-app'
    }

    stages {
        stage('Build') {
            steps {
                sh 'docker-compose build'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker-compose up -d
                    docker-compose ps
                    docker-compose logs app
                    docker compose exec -T app curl -v http://localhost:3000 || exit 1
                '''
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
                withCredentials([
                    usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS'),
                    sshUserPrivateKey(credentialsId: 'aws-ec2-ssh', keyFileVariable: 'KEY')
                ]) {
                    sh '''
                        chmod 600 "$KEY"
                        ssh -o StrictHostKeyChecking=no -i "$KEY" ubuntu@13.229.251.34 <<EOF
echo "$PASS" | docker login -u "$USER" --password-stdin
mkdir -p ~/devops-deploy && cd ~/devops-deploy

cat > docker-compose.yml <<EOL
services:
  app:
    image: technicalhrushi/devops-app
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=mydb
    depends_on:
      - db

  db:
    image: postgres:14
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=mydb
    volumes:
      - db_data:/var/lib/postgresql/data

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app

volumes:
  db_data:
EOL

mkdir -p nginx
cat > nginx/default.conf <<NGINX
server {
    listen 80;
    location / {
        proxy_pass http://app:3000;
    }
}
NGINX

docker compose down || true
docker compose up -d
EOF
                    '''
                }
            }
        }
    }
}
