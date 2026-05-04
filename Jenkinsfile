/*
 *  __________________________________________________________________________
 * |                                                                          |
 * |                    ENTERPRISE CI/CD PIPELINE v2.0                        |
 * |             ________________________________________________             |
 * |                                                                          |
 * |  Architect:  DevOps Automation Engine                                    |
 * |  Stack:      Jenkins | SonarQube | Docker | Node.js                      |
 * |  Purpose:    End-to-End Automated Quality & Deployment Lifecycle        |
 * |__________________________________________________________________________|
 */

pipeline {
    agent any

    environment {
        /* --- DOCKER CONFIGURATION --- */
        DOCKER_IMAGE = 'guru784/cicd-pipeline-app'
        DOCKER_TAG   = "v${env.BUILD_NUMBER}"
        
        /* --- TOOL DEFINITIONS --- */
        SONAR_SCANNER_HOME = tool 'SonarQubeScanner'
    }

    stages {
        
        // ====================================================================
        // STAGE 0: REPOSITORY SYNCHRONIZATION
        // ====================================================================
        stage('Repository Synchronization') {
            steps {
                script {
                    echo "Initializing Workspace for Build #${env.BUILD_NUMBER}"
                    checkout scm
                }
            }
        }

        // ====================================================================
        // STAGE 1: CONTINUOUS INTEGRATION - BUILD & UNIT TESTING
        // ====================================================================
        stage('Build & Test Execution') {
            steps {
                echo 'Phase 1: Installing Dependencies and executing Unit Tests...'
                sh """
                    npm install
                    npm test
                """
            }
        }

        // ====================================================================
        // STAGE 2: CONTINUOUS INSPECTION - SONARQUBE ANALYSIS & QUALITY GATE
        // ====================================================================
        stage('SonarQube Security & Quality Audit') {
            steps {
                echo 'Phase 2: Initiating Deep Static Analysis and Quality Gate verification...'
                /* 
                 * Using Synchronous Quality Gate Polling:
                 * We utilize -Dsonar.qualitygate.wait=true to force the pipeline to 
                 * halt if the security or quality thresholds are not satisfied.
                 */
                sh """
                    export SONAR_TOKEN='sqp_86b146665e364347b141e427893d0fde8e9cc2ed'
                    ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.host.url=http://sonarqube-server:9000 \
                        -Dsonar.qualitygate.wait=true
                """
            }
        }

        // ====================================================================
        // STAGE 3: IMMUTABLE PACKAGING - DOCKER CONTAINERIZATION
        // ====================================================================
        stage('Docker Artifact Generation') {
            steps {
                echo "Phase 3: Packaging Application into Immutable Docker Image: ${DOCKER_TAG}"
                sh """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                """
            }
        }

        // ====================================================================
        // STAGE 4: ARTIFACT DISTRIBUTION - DOCKERHUB REGISTRY PUSH
        // ====================================================================
        stage('Registry Distribution') {
            steps {
                echo 'Phase 4: Distributing validated artifacts to DockerHub...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh """
                        echo "\$DOCKER_PASS" | docker login -u \$DOCKER_USER --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        // ====================================================================
        // STAGE 5: CONTINUOUS DEPLOYMENT - LOCAL PRODUCTION RUNTIME
        // ====================================================================
        stage('Production Deployment (Local)') {
            steps {
                echo 'Phase 5: Deploying Container to Local Production-Ready Runtime...'
                sh """
                    docker stop app-container || true
                    docker rm app-container || true
                    docker run -d -p 3000:3000 --name app-container ${DOCKER_IMAGE}:latest
                """
            }
        }
    }

    // ========================================================================
    // POST-PIPELINE LIFECYCLE MANAGEMENT
    // ========================================================================
    post {
        always {
            echo 'Cleaning up authentication tokens and workspace state...'
            sh 'docker logout'
        }
        success {
            echo '============================================================'
            echo ' PIPELINE SUCCESSFUL: ARTIFACTS DEPLOYED & VERIFIED '
            echo '============================================================'
        }
        failure {
            echo '============================================================'
            echo ' PIPELINE FAILED: MANUAL INTERVENTION REQUIRED '
            echo '============================================================'
        }
    }
}
