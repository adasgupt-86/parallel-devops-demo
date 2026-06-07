pipeline {

    agent any

    tools {
        maven 'maven'
    }

    environment {
        IMAGE = "adasgupt86/parallel-demo"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/adasgupt-86/parallel-devops-demo.git',
                    credentialsId: 'git-cred'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build -t ${IMAGE}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Parallel Validation') {

            parallel {

                stage('Unit Test') {

                    steps {
                        sh 'mvn test'
                    }
                }

                stage('SonarQube Scan') {

                    steps {

                        withSonarQubeEnv('sonarqube') {

                            sh '''
                            mvn sonar:sonar \
                            -Dsonar.projectKey=parallel-demo
                            '''
                        }
                    }
                }

                stage('Trivy Scan') {

                    steps {

                        sh '''
                        trivy image \
                        ${IMAGE}:${BUILD_NUMBER}
                        '''
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
    }
}
