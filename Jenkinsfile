pipeline {

    agent any

    tools {
        jdk 'jdk21'
        nodejs 'Node26'
    }

    environment {
        IMAGE_NAME = 'likhidevops/zomato'
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Likhi1997/Docker-and-k8s-project/'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'

                    withSonarQubeEnv('sonar-server') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectName=zomato \
                            -Dsonar.projectKey=zomato-report \
                            -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
              /*   sh '''
                    trivy fs \
                    --severity HIGH,CRITICAL \
                    --ignore-unfixed \
                    --exit-code 1 \
                    .
                '''   */
                sh 'trivy fs . > trivy.txt'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                    -t ${IMAGE_NAME}:${IMAGE_TAG} \
                    -t ${IMAGE_NAME}:latest \
                    .
                '''
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                    trivy image \
                    --severity HIGH,CRITICAL \
                    --ignore-unfixed \
                    --exit-code 1 \
                    ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker') {
                        sh '''
                            docker push ${IMAGE_NAME}:${IMAGE_TAG}
                            docker push ${IMAGE_NAME}:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to Container') {
            steps {
                sh '''
                    docker rm -f zomato-container || true

                    docker run -d \
                    --name zomato-container \
                    -p 3000:80 \
                    ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {

        success {
            emailext(
                to: 'likhitha.kommana.70@gmail.com',
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    Jenkins build completed successfully.

                    Job: ${env.JOB_NAME}
                    Build: #${env.BUILD_NUMBER}
                    Image: ${IMAGE_NAME}:${IMAGE_TAG}
                    Status: SUCCESS
                """
            )
        }

        failure {
            emailext(
                to: 'likhitha.kommana.70@gmail.com',
                subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    Jenkins build failed.

                    Job: ${env.JOB_NAME}
                    Build: #${env.BUILD_NUMBER}
                    Status: FAILURE

                    Please check the Jenkins console output.
                """
            )
        }
    }
}
