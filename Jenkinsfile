pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        MAVEN_HOME = tool 'Maven_Home'
    }

    stages {
        stage('RunSCAAnalysisUsingSnyk') {
            steps {		
                echo 'Testing...'
                snykSecurity(
                    snykInstallation: 'synktool',
                    snykTokenId: 'snyk-token',
                    failOnError: 'false',
                    failOnIssues: 'false'
                )
            }
        }

        stage("Build") {
            steps {
                sh "${MAVEN_HOME}/bin/mvn clean install -DskipTests=true"
            }
        }
        
        stage("Sonar_scan") {
            steps {
                sh """
                ${MAVEN_HOME}/bin/mvn clean verify sonar:sonar \
                  -Dsonar.projectKey=petclinc-tes \
                  -Dsonar.projectName='petclinc-tes' \
                  -Dsonar.host.url=http://54.242.72.209:9000 \
                  -Dsonar.token=sqa_f7a08ee0d44d05d578ea7754c2d330841604c4d4
                """
            }
        }
                
        stage("Trivy Scan (Filesystem)") {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        
        stage("Image Build") {
            steps {
                sh "docker build -t promo286/petapp:${BUILD_NUMBER} ."
            }
        }
        
        stage("Trivy Scan (Docker Image)") {
            steps {
                sh "trivy image promo286/petapp:${BUILD_NUMBER} --scanners vuln > trivyimage.txt"
            }
        }
        
        stage("Docker Push") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    sh """
                    echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin
                    docker push promo286/petapp:${BUILD_NUMBER}
                    """
                }
            }
        }
    }
}
