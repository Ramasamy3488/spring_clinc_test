pipeline {
    agent any

    environment {
        SCANNER_HOME=tool 'sonar_scanner'
    }

    stages {
        stage("Build") {
            steps {
                sh "mvn clean install -DskipTests=true"
            }
        }
        
        stage("Sonar_scan") {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=devsecops28_petclinc28 \
                    -Dsonar.projectKey=devsecops28_petclinc28 \
                    -Dsonar.organization=devsecops28 \
                    -Dsonar.java.binaries=target/classes '''
                }
            }
        }
                
        stage("trivy scan") {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        
        stage("Image build") {
            steps {
                sh "docker build -t promo286/petapp:${BUILD_NUMBER} ."
            }
        }
        
        stage("TRIVY") {
            steps {
                sh "trivy image promo286/petapp:${BUILD_NUMBER} --scanners vuln > trivyimage.txt"
            }
        }
        
        stage("Docker Push") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    sh "echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin"
                    sh "docker push promo286/petapp:${BUILD_NUMBER}"
                }
            }
        }
    }
}
