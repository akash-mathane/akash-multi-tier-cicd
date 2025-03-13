pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'   // SonarQube Scanner tool name
        DOCKER_CREDENTIALS = credentials('docker-cred')  // DockerHub credentials
        KUBE_CONFIG = credentials('k8-cred')  // Kubernetes Service Account Token
        GIT_CRED = credentials('git-cred')  // GitHub Credentials
    }

    tools {
        maven 'maven3'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/akash-mathane/akash-multi-tier-cicd.git', credentialsId: 'git-cred'
            }
        }

        stage('Compile Code') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Static Code Analysis (SonarQube)') {
            steps {
                withSonarQubeEnv('sonar-token') { // Use SonarQube token
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=bankapp -Dsonar.sources=src"
                }
            }
        }

        stage('Build Package') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }

        stage('Publish Artifacts to Nexus') {
            steps {
                withMaven(maven: 'maven3', options: [configFileProvider('settings-maven')]) {
                    sh 'mvn deploy -DskipTests=true'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t skillfullsky/bankapp:latest .
                '''
            }
        }

        stage('Scan Docker Image with Trivy') {
            steps {
                sh '''
                    trivy image --format table --output trivy-report.html skillfullsky/bankapp:latest
                '''
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                withDockerRegistry([credentialsId: 'docker-cred', url: '']) {
                    sh 'docker push skillfullsky/bankapp:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'k8-cred']) {
                    sh 'kubectl apply -f k8s-deployment.yaml'
                }
            }
        }
    }
}
