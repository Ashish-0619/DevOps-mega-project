@Library('Shared')_

pipeline{
    agent any
    
    parameters{
        string(name: 'IMAGE_VERSION',defaultValue: 'latest', description: "Image tag")
    }
    environment{
        DOCKER_HUB_REPO = "rocish/mega-project"
        SONAR_PROJECT_KEY = "mega-project"
        SONAR_HOST_URL = "http://localhost:9000"
        SONAR_TOKEN = credentials('sonarqube')
    }
    stages{
        stage("Code"){
            steps{
                clone("https://github.com/Amitabh-DevOps/DevOps-mega-project.git","project")
                echo "Code clonning done."
            }
        }
        stage("Build"){                                                             
            steps{
                dockerbuild("mega-project","${params.IMAGE_VERSION}")
                echo "Code build bhi hogaya."
            }
        }
        stage("Push to DockerHub"){
            steps{
                dockerpush("dockerHub","mega-project","${params.IMAGE_VERSION}")
                echo "Push to dockerHub is also done."
            }
        }
    }
    stage("Static Code Analysis with SonarQube") {
            steps {
                echo "Running SonarQube analysis..."
                sh '''
                    sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.sources=project/src \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_TOKEN}
                    echo "SonarQube analysis completed."
                '''
            }
        }
        stage("Security Scan with Trivy") {
            steps {
                echo "Running security scan with Trivy..."
                sh '''
                    docker run --rm aquasec/trivy image ${DOCKER_HUB_REPO}:${params.IMAGE_VERSION} > security-scan-report.txt || true
                    echo "Security scan completed."
                '''
            }
        }
    }
    post {
        always {
            echo "Pipeline completed."
            archiveArtifacts artifacts: 'security-scan-report.txt', allowEmptyArchive: true
        }
        success {
            echo "All stages completed successfully."
        }
        failure {
            echo "Pipeline failed. Please check the logs for details."
        }
    }
}
