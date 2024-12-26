pipeline {
    agent any
    tools {
        maven "M2_HOME"
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials-id')
        MYSQL_CREDENTIALS = credentials('mysql-credentials')
    }
    stages {

        stage('MVN CLEAN') {
            steps {
                dir('kaddem') {
                    sh 'mvn clean'
                }
            }
        }

        stage('MVN COMPILE') {
            steps {
                dir('kaddem') {
                    sh 'mvn compile'
                }
            }
        }
        stage('MVN BUILD') {
            steps {
                dir('kaddem') {
                    sh 'mvn install -DskipTests=true'
                }
            }
        }

        
        stage('Build Backend Docker Image') {
            steps {
                dir('kaddem') {
                    sh 'docker build --no-cache -t kaddem-backend .'
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                    sh 'docker tag kaddem-backend ahmedmessaoudy/kaddem_project:$BUILD_NUMBER'
                    sh 'docker push ahmedmessaoudy/kaddem_project:$BUILD_NUMBER'
                }
            }
           }
        stage('Stop Old Containers') {
            steps {
                sh 'docker network create my-network || true'
                sh 'docker stop mysql_db || true && docker rm mysql_db || true'
                sh 'docker stop backend_ctr || true && docker rm backend_ctr || true'
                sh 'docker run -d --network my-network -p 3306:3306 --name mysql_db -e MYSQL_ROOT_PASSWORD=$MYSQL_CREDENTIALS_PSW mysql:latest'
                sh 'sleep 10'
                sh 'docker run -d --network my-network -p 8089:8089 --name backend_ctr ahmedmessaoudy/kaddem_project:$BUILD_NUMBER'

    }
    }


    }
}
