pipeline {
    agent any

    environment {
        // You MUST replace 'yourdockerhubusername' with your actual DockerHub username
        DOCKER_IMAGE = 'guru/cicd-pipeline-app'
        DOCKER_TAG = "v${env.BUILD_NUMBER}"
        SONAR_SCANNER_HOME = tool 'SonarQubeScanner'
    }

    stages {
        stage('Checkout') {
            steps {
                // Jenkins will automatically checkout the code from the webhook trigger, 
                // but if using SCM explicitly:
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                echo 'Installing dependencies and running tests...'
                sh 'npm install'
                sh 'npm test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube static code analysis...'
                withSonarQubeEnv('SonarQube') { 
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh "${SONAR_SCANNER_HOME}/bin/sonar-scanner -Dsonar.login=\$SONAR_TOKEN"
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Wait for SonarQube webhook to return Quality Gate status
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker Image...'
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
            }
        }

        stage('Docker Push') {
            steps {
                echo 'Pushing image to DockerHub...'
                // 'dockerhub-credentials' is the ID of the credentials you must add in Jenkins
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Deploy Locally') {
            steps {
                echo 'Deploying container locally...'
                // Stop and remove the old container if it exists
                sh 'docker stop app-container || true'
                sh 'docker rm app-container || true'
                // Run the new container
                sh "docker run -d -p 3000:3000 --name app-container ${DOCKER_IMAGE}:latest"
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
            sh 'docker logout'
        }
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Pipeline failed. Check the logs.'
        }
    }
}
