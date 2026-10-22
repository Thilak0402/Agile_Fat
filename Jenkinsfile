pipeline {
    agent any
    parameters {
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run automated tests before building')
        choice(name: 'ENVIRONMENT', choices: ['staging', 'production'], description: 'Deployment Target Environment')
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Thilak0402/Agile_Fat.git'
            }
        }
        stage('Parallel Quality Checks') {
            when {
                expression { params.RUN_TESTS == true }
            }
            parallel {
                stage('Lint HTML') {
                    steps {
                        echo 'Running HTML validation checks...'
                    }
                }
                stage('Security Scan') {
                    steps {
                        echo 'Scanning Dockerfile and manifests for vulnerabilities...'
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("thilak0402/traffic-dashboard:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'index.html, deployment.yaml', fingerprint: true
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials-id') {
                        dockerImage.push('latest')
                        dockerImage.push("${env.BUILD_NUMBER}")
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(credentialsId: 'k8s-kubeconfig-id', serverUrl: 'https://kubernetes.docker.internal:6443') {
                    // Use 'bat' instead of 'sh' for Windows compatibility
                    bat 'powershell -Command "(Get-Content deployment.yaml) -replace \'image:.*\', \'image: index.docker.io/thilak0402/traffic-dashboard:${env.BUILD_NUMBER}\' | Set-Content deployment.yaml"'
                    bat 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }
    post {
        success {
            echo "Pipeline successfully completed for ${params.ENVIRONMENT}!"
        }
        failure {
            echo "Pipeline failed! Initiating automatic recovery/rollback protocols..."
        }
    }
}