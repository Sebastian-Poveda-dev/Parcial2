#!/usr/bin/env groovy

pipeline {
    agent any
    options {
        timestamps()
        skipDefaultCheckout(true)
    }
    environment {
        APP_NAME = "cicd-demo"
        IMAGE_TAG = "${APP_NAME}:${BUILD_NUMBER}"
        DEPLOY_CONTAINER = "${APP_NAME}-local"
        SONARQUBE_SERVER = "sonarqube"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh "chmod +x mvnw"
                sh "./mvnw -q -DskipTests clean package"
            }
        }
        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_TAG} ."
            }
        }
        stage('Test') {
            steps {
                sh "./mvnw -q test -Dgroups=UnitTest"
            }
        }
        stage('Static Analysis (SonarQube)') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh "./mvnw -q -DskipTests sonar:sonar -Dsonar.projectKey=${APP_NAME}"
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Container Security Scan (Trivy)') {
            steps {
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.52.2 image --no-progress --severity CRITICAL --exit-code 1 ${IMAGE_TAG}"
            }
        }
        stage('Deploy') {
            when { branch 'master' }
            steps {
                sh "docker rm -f ${DEPLOY_CONTAINER} || true"
                sh "docker run -d --name ${DEPLOY_CONTAINER} -p 80:8080 ${IMAGE_TAG}"
            }
        }
    }
    post {
        failure {
            echo 'Pipeline failed. Check the stage logs for details.'
        }
        always {
            deleteDir()
        }
    }
}
