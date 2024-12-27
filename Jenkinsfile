pipeline {
    agent any
    tools {
        maven "M2_HOME"
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials-id')
        MYSQL_CREDENTIALS = credentials('mysql-credentials')
        NEXUS_VERSION = 'nexus3'
        NEXUS_PROTOCOL = 'http'
        NEXUS_URL = '192.168.50.3:8081' // Nexus server base URL
        NEXUS_REPOSITORY = 'tp-foyernexus'  // Nexus repository name
        CREDENTIALS_ID = 'nexus-cred'      // Jenkins credentials ID for Nexus
        GROUP_ID = 'tn.esprit'  // Group ID for the artifact
    }
    stages {
         stage("Maven Build") {
            steps {
                 
                   sh "mvn package -DskipTests=true"
                
                
            }
        }

       
       

          stage("Upload to Nexus") {
            steps {
                
                script {
                    def artifactId = 'tp-foyer'  // Define your artifact ID
                    def version = '5.0.0'                  // Define the version
                    def packaging = 'jar'                // Define the packaging type (e.g., jar)

                    // Correct usage of nexusArtifactUploader
                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: GROUP_ID,
                        artifactId: artifactId,
                        version: version,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: CREDENTIALS_ID,
                        artifacts: [
                            [artifactId: artifactId,
                             classifier: '',
                             file: "target/${artifactId}-${version}.${packaging}",
                             type: packaging]
                        ]
                    )
                
            }
            }
        }
        stage('Compile') {
            steps {
                
                    sh 'mvn clean compile'
                
            }
        }
      
        
        stage('Junit+Mock') {
            steps {
                
                    sh 'mvn test'
                
            }
        }
        stage('SonnarQube') {
            steps {
                
                    withSonarQubeEnv('sq1') {
                        sh 'mvn sonar:sonar -Dsonar.java.binaries=target/classes'
                    }
                
            }
        } 
       /*  stage('Build Backend Docker Image') {
            steps {
                    sh 'docker build -t kaddem-backend .'
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                    sh 'docker tag kaddem-backend ahmedmessaoudy/kaddem_project:$BUILD_NUMBER'
                    sh 'docker push ahmedmessaoudy/kaddem_project:$BUILD_NUMBER'
            }
           }
           */
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
