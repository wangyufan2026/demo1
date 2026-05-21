pipeline {
    agent any

    tools {
        maven 'maven 3.9'
        jdk 'jdk 25'
    }

    environment {
        GIT_REPO = 'git@github.com:wangyufan2026/demo1.git'
        GIT_BRANCH = 'main'
        GIT_CREDENTIALS_ID = 'github-ssh-key'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub via SSH...'
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${GIT_BRANCH}"]],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [[$class: 'CleanBeforeCheckout']],
                    userRemoteConfigs: [[
                        url: "${GIT_REPO}",
                        credentialsId: "${GIT_CREDENTIALS_ID}"
                    ]]
                ])
            }
        }

        stage('Build') {
            steps {
                echo 'Compiling the Spring Boot application...'
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh 'mvn test'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package JAR') {
            steps {
                echo 'Packaging the application into an executable JAR...'
                sh 'mvn package -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, onlyIfSuccessful: true
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! Build #${env.BUILD_NUMBER}"
        }
        failure {
            echo "Pipeline failed at build #${env.BUILD_NUMBER}"
        }
        always {
            echo 'Cleaning up workspace...'
        }
    }
}