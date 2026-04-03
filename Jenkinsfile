pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node18'
    }

    environment {
        DOCKER_IMAGE = "kamales113/zomato"
        SCANNER_HOME = tool 'sonar-scanner'
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
                    url: 'https://github.com/Kamales113/Zomato-CI-CD.git'
            }
        }

        // ✅ SAFE SONARQUBE ADDITION
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: false
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'docker build -t zomato .'
                    }
                }
            }
        }

        stage('Tag & Push to DockerHub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh '''
                        docker tag zomato ${DOCKER_IMAGE}:latest
                        docker push ${DOCKER_IMAGE}:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f Kubernetes/deployment.yaml
                kubectl apply -f Kubernetes/service.yaml
                kubectl apply -f Kubernetes/hpa.yaml
                kubectl rollout status deployment/zomato
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully. App deployed to EKS.'
        }
        failure {
            echo '❌ Pipeline failed. Check logs.'
        }
    }
}
