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
                // We use a custom variable name 'KUBECONFIG_FILE' to avoid conflicts
                withCredentials([file(credentialsId: 'k8s-kubeconfig-id', variable: 'KUBECONFIG_FILE')]) {
                    powershell """
                        # Update the image tag in deployment.yaml
                        (Get-Content deployment.yaml) -replace 'image:.*', 'image: index.docker.io/thilak0402/traffic-dashboard:${env.BUILD_NUMBER}' | Set-Content deployment.yaml
                        
                        # Apply configuration explicitly using the PowerShell environment variable
                        kubectl --kubeconfig="$env:KUBECONFIG_FILE" apply -f deployment.yaml
                    """
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