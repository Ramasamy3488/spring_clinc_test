pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        MAVEN_HOME = tool 'Maven_Home'
    }

    stages {

        stage('Run SCA Analysis Using Snyk') {
            steps {
                echo 'Testing...'
                sh "chmod +x mvnw"
                snykSecurity(
                    snykInstallation: 'snyk-tool',
                    snykTokenId: 'snyk-token',
                    failOnError: false,
                    failOnIssues: false
                )
            }
        }

        stage("Build") {
            steps {
                sh "mvn clean install -DskipTests=true"
            }
        }

        stage("Sonar Scan") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                    ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=suryateck-project_petclinic \
                        -Dsonar.organization=suryateck-project \
                        -Dsonar.projectName=petclinic \
                        -Dsonar.language=java \
                        -Dsonar.sourceEncoding=UTF-8 \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                    """
                }
            }
        }

        stage("Trivy Scan (Filesystem)") {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage("Image Build") {
            steps {
                sh "docker build -t ramasamy123/petapp:${BUILD_NUMBER} ."
            }
        }

        stage("Trivy Scan (Docker Image)") {
            steps {
                sh """
                trivy image ramasamy123/petapp:${BUILD_NUMBER} --scanners vuln --format table --output trivyimage_table.txt
                trivy image ramasamy123/petapp:${BUILD_NUMBER} --scanners vuln --format json --output trivyimage.json
                """
            }
        }

        stage("Docker Push") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    sh """
                    echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin
                    docker push ramasamy123/petapp:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage("Docker Deploy") {
            steps {
                sh "docker run -d -it --name demo -p 9000:8000 ramasamy123/petapp:${BUILD_NUMBER}"
            }
        }
    }
}
