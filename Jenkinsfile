pipeline {
    agent any

    environment {
        PATH = "/usr/local/bin:${env.PATH}"
        KUBECONFIG = "/Users/geraldoti/kubeconfig-cred/config" 
    }

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Target environment')
        choice(name: 'ACTION', choices: ['build', 'deploy'], description: 'Action to perform')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Kustomize') {
            steps {
                sh """
                    echo "Validating Kustomize configuration for ${params.ENVIRONMENT}"
                    kustomize build overlays/${params.ENVIRONMENT} --load-restrictor LoadRestrictionsNone > /dev/null
                    echo "✓ Kustomize validation successful"
                """
            }
        }

        stage('Build Manifests') {
            steps {
                sh """
                    set -e
                    echo "Building Kustomize manifests for ${params.ENVIRONMENT}"
                    kustomize build overlays/${params.ENVIRONMENT} --load-restrictor LoadRestrictionsNone > build-output.yaml
                    echo "✓ Build completed successfully"
                """
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                expression { params.ACTION == 'deploy' }
            }
            steps {
                sh """
                    set -e
                    echo "Deploying to ${params.ENVIRONMENT} cluster"
                    # Create namespace if it doesn't exist (simpler approach)
                    kubectl create namespace ${params.ENVIRONMENT} --dry-run=client -o yaml | kubectl apply --validate=false -f -
                    # Apply with explicit namespace
                    kubectl apply -n ${params.ENVIRONMENT} -f build-output.yaml
                    echo "✓ Deployment completed successfully"
                    
                    echo "Deployment status:"
                    kubectl get deployments -n ${params.ENVIRONMENT}
                    kubectl get services -n ${params.ENVIRONMENT}
                """
            }
        }

        stage('Verify Deployment') {
            when {
                expression { params.ACTION == 'deploy' }
            }
            steps {
                sh """
                    echo "Verifying deployment..."
                    kubectl rollout status deployment/web-app-${params.ENVIRONMENT} -n ${params.ENVIRONMENT} --timeout=120s
                    echo "✓ Deployment verified successfully"
                """
            }
        }
    }

    post {
        always {
            sh 'rm -f build-output.yaml || true'
        }
        success {
            echo "Pipeline executed successfully for ${params.ENVIRONMENT}"
        }
        failure {
            echo "Pipeline failed for ${params.ENVIRONMENT}"
        }
    }
}
