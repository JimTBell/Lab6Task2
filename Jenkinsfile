pipeline {
    agent any
    environment {
        DOCKERHUD = credentials('docker_credentials')
        SQL_PASSWORD='my-secret-pd'
    }
    stages {
        stage('Cleanup Stage') {
            steps {
                sh "docker stop  \$(docker ps -q)  || echo \"No containers to stop\""
                sh "docker rm -f \$(docker ps -a -q) || echo \"No containers removed\""
                sh "docker rmi -f \$(docker images -q) || echo \"No images removed\""
                sh "docker network rm lab9-network || true"
            }
        }
        stage('Build Stage') {
            steps {
                sh "docker network create lab9-network"
                sh "docker build -t jimtbell/dba db"
                sh "docker build -t jimtbell/flask-app flask-app"

                sh "docker run --name mysql --network lab9-network -e \"MYSQL_ROOT_PASSWORD=${SQL_PASSWORD}\" -d jimtbell/dba"
                sh "docker run --name flask-app --network lab9-network -d -e\"MYSQL_ROOT_PASSWORD=${SQL_PASSWORD}\" jimtbell/flask-app"
                sh "docker run -d -p 80:80 --name nginx --network lab9-network --mount type=bind,source=\"\$(pwd)\"/nginx/nginx.conf,target=/etc/nginx/nginx.conf nginx"
            }
        }
        stage('Test') {
            steps {
                sh "docker images"
            }
        }
        stage('Deploy') {
            steps {
                sh "echo \$DOCKERHUD_PSW | docker login -u \$DOCKERHUD_USR --password-stdin"
                sh "docker tag jimtbell/dba jimtbell/dba:latest"
                sh "docker tag jimtbell/flask-app jimtbell/flask-app:latest"
                sh "docker tag nginx jimtbell/nginx:latest"
                sh "docker push jimtbell/dba:latest"
                sh "docker push jimtbell/flask-app:latest"
                sh "docker push jimtbell/nginx:latest"
            }
        }
    }
}
