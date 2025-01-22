@Library('Shared')_

pipeline{
    agent any
    
    parameters{
        string(name: 'IMAGE_VERSION',defaultValue: 'latest', description: "Image tag")
    }
    environment{
        DOCKER_HUB_REPO = "rocish/mega-project"
        SONAR_PROJECT_KEY = "mega-project"
        SONAR_TOKEN = credentials('Jenkins-sonarqube')
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

        stage("Static Code Analysis with SonarQube") {
            steps {
                echo "Running SonarQube analysis..."
                withSonarQubeEnv('SonarQubeServer') {
                    sh '''
                        sonar-scanner \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.sources=/var/lib/jenkins/workspace/DevOps-mega-project/src/ \
                            -Dsonar.java.binaries=. \
                            -Dsonar.login=${SONAR_TOKEN}
                        echo "SonarQube analysis completed."
                    '''
                }
            }
        }
        stage("Security Scan with Trivy") {
            steps {
                echo "Running security scan with Trivy..."
                script {
                    // Use bash explicitly by setting the shell to bash
                    sh '''#!/bin/bash
                        echo "Using bash shell: $(which bash)"
                        echo "DOCKER_HUB_REPO: ${DOCKER_HUB_REPO}"
                        echo "IMAGE_VERSION: ${params.IMAGE_VERSION}"
                        docker run --rm aquasec/trivy image ${DOCKER_HUB_REPO}:${params.IMAGE_VERSION} > security-scan-report.txt || true
                        echo "Security scan completed."
                    '''
                }
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
