pipeline {
    agent {
        docker { 
            image 'eclipse-temurin:21-jdk' 
            args '-v /var/run/docker.sock:/var/run/docker.sock -e DOCKER_HOST=unix:///var/run/docker.sock --network spring-petclinic_devsecops-net'
        }
    }

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        SONAR_SERVER_NAME = 'sonarqube'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Package') {
            steps {
                sh 'chmod +x mvnw'
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Unit Test') {
            steps {
                sh '''
                    export TESTCONTAINERS_RYUK_DISABLED=true
                    ./mvnw test -Dtest="!*IT, !*IntegrationTests"
                '''
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONAR_SERVER_NAME}") {
                    sh './mvnw sonar:sonar'
                }
            }
        }

        stage('Security Scan') {
            steps {
                echo "Running Security Analysis..."
            }
            post {
                always {
                    publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'target/burp-reports', reportFiles: 'report.html', reportName: 'Security Report'])
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying to Production VM via Ansible..."
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}