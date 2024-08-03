pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                script {
                    sh 'mvn clean deploy -Dmaven.test.skip=true'
                }
            }
        }

        stage('Sonar Scanner') {
            environment {
                scannerHome = tool 'sonar-tool'
            }
            when { expression { params.action == 'Build' } }
            steps {
                script {
                    withSonarQubeEnv('sonar-server') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=promoth-28_jenkins-ci \
                            -Dsonar.organization=promoth-28 \
                            -Dsonar.projectName=Jenkins-ci \
                            -Dsonar.language=java \
                            -Dsonar.sourceEncoding=UTF-8 \
                            -Dsonar.sources=. \
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                        """
                    }
                }
            }
        }

        stage('Quality Gates') {
            when { expression { params.action == 'Build' } }
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    app = docker.build(imageName + ":" + version)
                }
            }
        }
    }
}
