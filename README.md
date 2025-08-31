# Capstone-kustomise

Hypothetical Use Case: You are tasked with deploying a web application in a Kubernetes environment. The application will have different configurations for development, staging, and production environments. Your goal is to utilize Kustomize to manage these configurations efficiently and integrate the process into a
CI/CD pipeline.

Tasks: Task 1: Set Up Your Project:
Create a new project directory named 'kustomize-capstone'.

Inside the directory, structure it with Kustomize conventions: *base/' and overlays/' (subdirectories for 'dev', 'staging', "prod).

Task 2: Initialize Git:

Initialize a Git repository in your 'kustomize-capstone' directory.

Create a ' gitignore' file to exclude unnecessary files.

Task 3: Define Base Configuration:

In the base/' directory, define Kubernetes resources like Deployment, Service, etc., for your web application.

Create a 'kustomization, yaml' file in 'base/' to include these resources.

Task 4: Create Environment-Specific Overlays:

In each subdirectory of overlays/', create a 'kustomization, yaml' that customizes the base configuration for that environment.

Implement variations for each environment (e.g., different replica counts, resource limits, or environment variables).

Task 5: Integrate with a CI/CD Pipeline:

Choose a Cl/CD platform (e.g., GitHub Actions, Jenkins).

Set up a pipeline that deploys your application using Kustomize. The pipeline should trigger on code changes.

Task 6: Test the Cl/CD Pipeline:

Make changes in your Kustomize configurations and push to your repository.

Verify that the CI/CD pipeline correctly applies these changes to a Kubernetes cluster.

Task 7: Manage Secrets and ConfigMaps:

Use Kustomize to generate ConfigMaps and Secrets. Ensure sensitive data is handled securely.

Apply these configurations in your overlays for different environments.

Task 8: Document Your Work:

Create a 'README. md in your project explaining the structure and how to deploy the application using Kustomize.

Include instructions for setting up and testing the CI/CD pipeline.

Task 9 (Advanced): Implement Transformers and Generators:

Use advanced features of Kustomize such as transformers and generators to refine your configurations.

Demonstrate usage like adding common labels, annotations, and managing dynamic data with generators.

Evaluation Criteria:
Correct implementation of Kustomize features across different environments.

Successful integration and functionality of the CI/CD pipeline.

Adherence to best practices in managing Kubernetes configurations, especially for secrets.

Clarity and completeness of documentation.


--------------------------------------------------------------------------------------------------------------------------------------------

                                                        PROJECT IMPLEMENTATION

--------------------------------------------------------------------------------------------------------------------------------------------
# Kustomize Capstone Project

## Overview

This project demonstrates how to use Kustomize for managing Kubernetes deployments across multiple environments (dev, staging, prod) with a CI/CD pipeline integration.


## Prerequisites
- Kubernetes cluster
- kubectl
- kustomize
- Jenkins (for CI/CD)

## Project Structure


```

kustomize-capstone/
├── base/
│   ├── common-labels.yaml
│   ├── deployment.yaml
│   ├── kustomization.yaml
│   └── service.yaml
├── overlays/
│   ├── dev/
│   │   ├── config.env.example
│   │   ├── kustomization.yaml
│   │   └── secret.env.example
│   ├── staging/
│   │   ├── kustomization.yaml
│   │   └── patch.yaml
│   └── prod/
│       ├── kustomization.yaml
│       └── patch.yaml
├── Jenkinsfile
├── .gitignore
└── README.md

```

# Task 1: Project Setup

```

mkdir kustomize-capstone
cd kustomize-capstone
mkdir -p base overlays/{dev,staging,prod}

```

# Task 2: Git Initialization

```
git init

```

.gitignore:

```

cat > .gitignore << 'EOF'
# Kustomize build artifacts
*.log
build-output.yaml  # Only exclude generated files, not all YAML

# Local env files (never commit secrets)
**/secret.env
**/config.env

# macOS & editor junk
.DS_Store
.idea/
.vscode/

# Kubernetes
.kube/

# Temporary files
tmp/
temp/

# IDE specific files
*.swp
*.swo
*~
EOF


```


# Task 3: Base Configuration

base/deployment.yaml:

```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
        - name: web-app
          image: nginx:alpine
          ports:
            - containerPort: 80
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "128Mi"
              cpu: "500m"

```

base/service.yaml:


```

apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP

```

base/kustomization.yaml:

```

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml

labels:
  - pairs:
      app.kubernetes.io/part-of: kustomize-capstone
      app.kubernetes.io/managed-by: kustomize
    includeSelectors: true
    includeTemplates: true

```

# Task 4: Environment-Specific Overlays

overlays/dev/kustomization.yaml:

```

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

nameSuffix: -dev
namespace: dev

replicas:
  - name: web-app
    count: 1

configMapGenerator:
  - name: app-config
    behavior: create
    literals:
      - ENVIRONMENT=development
      - LOG_LEVEL=debug
      - FEATURE_FLAG=alpha

secretGenerator:
  - name: app-secrets
    behavior: create
    literals:
      - DB_PASSWORD=devpassword123
      - API_KEY=dev-api-key-456

patches:
  - target:
      kind: Deployment
      name: web-app
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0/resources/limits/memory
        value: 256Mi
      - op: replace
        path: /spec/template/spec/containers/0/resources/requests/memory
        value: 128Mi

```

overlays/staging/kustomization.yaml:

```

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

nameSuffix: -staging
namespace: staging

replicas:
  - name: web-app
    count: 3

configMapGenerator:
  - name: app-config
    behavior: create
    literals:
      - ENVIRONMENT=staging
      - LOG_LEVEL=info
      - FEATURE_FLAG=beta

patches:
  - path: patch.yaml

```

overlays/staging/patch.yaml:

```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  template:
    spec:
      containers:
        - name: web-app
          envFrom:
            - configMapRef:
                name: app-config
          resources:
            limits:
              memory: "512Mi"
              cpu: "1000m"

```

overlays/prod/kustomization.yaml:

```

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

nameSuffix: -prod
namespace: prod

replicas:
  - name: web-app
    count: 5

configMapGenerator:
  - name: app-config
    behavior: create
    literals:
      - ENVIRONMENT=production
      - LOG_LEVEL=warn
      - FEATURE_FLAG=stable

patches:
  - path: patch.yaml

```

overlays/prod/patch.yaml:

```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  template:
    spec:
      containers:
        - name: web-app
          envFrom:
            - configMapRef:
                name: app-config
          resources:
            limits:
              memory: "1Gi"
              cpu: "2000m"
            requests:
              memory: "512Mi"
              cpu: "1000m"

```


# Task 5: CI/CD Pipeline Integration (Jenkins)


Jenkinsfile:


```

pipeline {
    agent any

    environment {
        PATH = "/usr/local/bin:${env.PATH}"
        KUBECONFIG = "/Users/geraldoti/.kube/config" 
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
```
🔹 Step 1: Install Jenkins & Plugins

On your Jenkins server:

1. Install Jenkins (if not already installed).

2. Install these plugins:

	- Pipeline (for Jenkinsfile pipelines)

	- Git (for SCM checkout)

	- Kubernetes CLI Plugin (optional, if you want kubectl support inside Jenkins)

	- Credentials Binding Plugin



🔹 Step 2: Add Kubernetes Credentials

We need to allow Jenkins to connect to your cluster.

1. Get your kubeconfig file:


```

cat ~/.kube/config


```


2. In Jenkins UI → Manage Jenkins → Credentials → Global Credentials

3. Add new Secret file or Secret text:

	- ID: kubeconfig-cred (this must match what we used in the Jenkinsfile).

	- Paste kubeconfig contents.


🔹 Step 3: Create Jenkins Pipeline Job

1. In Jenkins UI → New Item → Name: kustomize-deployment-pipeline.

2. Select Pipeline → OK.

4. Scroll to Pipeline section:


	- Definition: Pipeline script from SCM

	- SCM: Git

	- Repo URL: your GitHub/GitLab repo URL

	- Branch: main (or whichever branch you want)

	- Script Path: Jenkinsfile



🔹 Step 4: Trigger the Pipeline

You can trigger manually or automatically:

Option A: Manual Trigger

1. Go to your pipeline in Jenkins.

2. Click Build with Parameters.

3. Choose environment: dev, staging, or prod.

4. Jenkins runs the pipeline:

	- Builds manifests with Kustomize

	- Applies to cluster

	- Verifies rollout


Option B: Auto Trigger (Git Webhook)

1. In your GitHub repo → Settings → Webhooks.

2. Add webhook:

URL: http://<jenkins-server>:8080/github-webhook/

3. Content type: application/json

4. Trigger: push events

5. In Jenkins pipeline config, enable GitHub hook trigger for GITScm polling.




# Task 7: Secrets and ConfigMaps Management

Example environment files (not committed to Git):

overlays/dev/config.env.example:

```

ENVIRONMENT=development
LOG_LEVEL=debug
DATABASE_URL=dev-db-url
API_ENDPOINT=https://dev-api.example.com

```

overlays/dev/secret.env.example:

```

DB_PASSWORD=dev-secret-password
API_KEY=dev-api-key-secret
JWT_SECRET=dev-jwt-secret-key

```


## Deployment

### Local Testing
```bash
# Build dev environment
kustomize build overlays/dev

# Build staging environment  
kustomize build overlays/staging

# Build production environment
kustomize build overlays/prod

```

Jenkins Pipeline

Set up Jenkins with Kubernetes credentials

Create a pipeline job using the Jenkinsfile

Configure the job parameters:

- ENVIRONMENT: dev/staging/prod

- ACTION: build/deploy

Manual Deployment

```

# Apply dev configuration
kustomize build overlays/dev | kubectl apply -f -

# Apply staging configuration
kustomize build overlays/staging | kubectl apply -f -

# Apply production configuration  
kustomize build overlays/prod | kubectl apply -f -

```

Environment Differences

- Dev: 1 replica, debug logging, lower resources
  
- Staging: 3 replicas, info logging, medium resources
  
- Production: 5 replicas, warn logging, high resources

Security Notes

- Never commit actual secrets to Git

- Use Jenkins credentials for production secrets

- Example files (.example) are provided as templates


# Task 9: Advanced Features

**base/common-labels.yaml:**

```yaml
apiVersion: builtin
kind: LabelTransformer
metadata:
  name: common-labels
labels:
  app.kubernetes.io/name: web-app
  app.kubernetes.io/instance: "{{.Environment}}"
  app.kubernetes.io/version: "1.0.0"
  app.kubernetes.io/component: frontend
  app.kubernetes.io/part-of: kustomize-capstone
  app.kubernetes.io/managed-by: kustomize
fieldSpecs:
  - path: metadata/labels
    create: true
  - path: spec/selector/matchLabels
    create: true
  - path: spec/template/metadata/labels
    create: true
```

---------------------------------------------------------------------------------------------------------------------------------------

                                              IMPLEMENTATION IN ACTION
--------------------------------------------------------------------------------------------------------------------------------------


<img width="1276" height="176" alt="image" src="https://github.com/user-attachments/assets/39006a86-d708-4992-ac65-0feffe96216a" />


<img width="2420" height="404" alt="image" src="https://github.com/user-attachments/assets/67e4f12f-62ab-4271-84c0-d58e98173300" />


<img width="2306" height="1108" alt="image" src="https://github.com/user-attachments/assets/7ccb7cb3-6365-4bcc-986d-917977ce3354" />


<img width="2214" height="760" alt="image" src="https://github.com/user-attachments/assets/0e6cd1f0-137a-4508-88cb-372551a9632a" />


<img width="2372" height="1106" alt="image" src="https://github.com/user-attachments/assets/d71671cd-38ec-4a2e-a48f-310b951d9037" />


<img width="1908" height="386" alt="image" src="https://github.com/user-attachments/assets/3456e59c-af69-4d87-9f94-b35df7014dc8" />



<img width="2390" height="132" alt="image" src="https://github.com/user-attachments/assets/1e55cf8d-8fc4-4f51-89d4-88c4f235c84a" />


<img width="1424" height="606" alt="image" src="https://github.com/user-attachments/assets/ffa55d66-b0b3-45ef-ab36-2ee0a943252d" />


<img width="1410" height="770" alt="image" src="https://github.com/user-attachments/assets/4f5c9e82-4dfb-46fa-a5b3-b5e41e2d3b29" />


<img width="1414" height="764" alt="image" src="https://github.com/user-attachments/assets/d16194de-2848-4c72-8306-39e222053a5f" />


<img width="1418" height="581" alt="image" src="https://github.com/user-attachments/assets/10b7282d-b34e-4e0c-87a9-cc04d9ebb89d" />


<img width="2858" height="1556" alt="image" src="https://github.com/user-attachments/assets/a53870ba-590c-4ad7-9312-f5aeed588539" />


<img width="2846" height="1532" alt="image" src="https://github.com/user-attachments/assets/5217d873-971a-4292-a311-46e622f39913" />


<img width="2860" height="1528" alt="image" src="https://github.com/user-attachments/assets/02966e4a-4487-433b-93b1-acbac59c61d9" />


<img width="2836" height="1538" alt="image" src="https://github.com/user-attachments/assets/265103a8-3ae4-47ef-a31a-93d0c2b9a33e" />


<img width="2818" height="1368" alt="image" src="https://github.com/user-attachments/assets/aa2962f5-d2df-477b-9e04-a5daec1b69f0" />


<img width="1796" height="144" alt="image" src="https://github.com/user-attachments/assets/fc0880ed-f084-4652-b000-dfd33fc12d9e" />


<img width="1435" height="610" alt="image" src="https://github.com/user-attachments/assets/e3e770ca-0b51-443a-8fbe-6bfd4d9b0b88" />


<img width="2868" height="1548" alt="image" src="https://github.com/user-attachments/assets/5eccf1aa-dc3f-4ce1-a91e-b699087fa9f2" />


<img width="2044" height="1416" alt="image" src="https://github.com/user-attachments/assets/b525119a-b971-4a53-b9ec-b83bfce16c15" />


<img width="2074" height="1426" alt="image" src="https://github.com/user-attachments/assets/1e319876-1ad0-4d09-aefc-1382056a1335" />


<img width="2128" height="1412" alt="image" src="https://github.com/user-attachments/assets/6b58c9f2-8664-4058-b939-1987880cef65" />


<img width="2852" height="1372" alt="image" src="https://github.com/user-attachments/assets/8f80e978-53d8-4b78-958e-e2b547cbc6fb" />


<img width="2104" height="350" alt="image" src="https://github.com/user-attachments/assets/629ad190-93aa-4275-80f7-bf41fb23c8d5" />


<img width="1435" height="506" alt="image" src="https://github.com/user-attachments/assets/f316bdb0-25c7-4fd5-9dc2-6a5b26a32c85" />


<img width="2838" height="1546" alt="image" src="https://github.com/user-attachments/assets/b4fb94d7-3c54-4f44-9b80-a2e4224667c2" />


<img width="2816" height="1544" alt="image" src="https://github.com/user-attachments/assets/60d522b5-b708-4833-b6d0-f9ccdf5cf367" />


<img width="2860" height="1458" alt="image" src="https://github.com/user-attachments/assets/f0c88cee-6be7-4423-9716-f450d313b512" />


<img width="2032" height="314" alt="image" src="https://github.com/user-attachments/assets/bba13752-f45c-4e03-a504-80d446927b0b" />


changing replica counts for dev from 1 to 2

<img width="1017" height="470" alt="image" src="https://github.com/user-attachments/assets/db47cd58-d0e7-43ce-8183-fbe18d125965" />


<img width="2156" height="530" alt="image" src="https://github.com/user-attachments/assets/b4340cd9-ca9e-49e0-b9f1-d6ca371ce143" />


<img width="1401" height="495" alt="image" src="https://github.com/user-attachments/assets/b1061619-cce7-4739-8b99-99a3ac482103" />

<img width="2874" height="1550" alt="image" src="https://github.com/user-attachments/assets/5ba32808-f73b-4898-b55f-7dc922a34cee" />


<img width="2842" height="1548" alt="image" src="https://github.com/user-attachments/assets/402dfe8c-7a93-4396-b4bf-c16ab59f6e3a" />


<img width="2126" height="298" alt="image" src="https://github.com/user-attachments/assets/87e44524-99f8-40ec-a2ab-bf899ef38bc0" />










































