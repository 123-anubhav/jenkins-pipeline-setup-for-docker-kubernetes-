 Jenkins pipeline and the provided script.

---

# **Jenkins Declarative Pipeline for Docker and Kubernetes**

This Jenkins pipeline automates the process of building, testing, and deploying a Maven-based web application using Docker and Kubernetes. It includes several stages to ensure a streamlined CI/CD workflow.

---

## **Pipeline Stages**

### 1. **Git Clone**
- Fetches the source code from the specified GitHub repository.
- Ensures the latest changes are always pulled before the build process.

### 2. **Maven Clean and Package**
- Cleans the Maven workspace and compiles the project.
- Packages the application into a `.jar` or `.war` file, depending on the project configuration.

### 3. **Docker Image Creation**
- Builds a Docker image using the application code and the provided `Dockerfile`.
- Tags the image with the name `anubhav`.

### 4. **Set Kubernetes Context**
- Checks if the `docker-desktop` context exists in the Kubernetes configuration.
- If the context is found, it switches to it; otherwise, it logs a message that the context is unavailable.

### 5. **Kubernetes Deployment**
- Applies the Kubernetes deployment configurations using the `kubectl apply` command.
- Reads the deployment files from the specified directory (`YAML_PATH`).

---

## **Pipeline Configuration**

Below is the declarative pipeline script used in this project:

```groovy
pipeline {
    agent any

    tools {
        maven 'm3'
    }

    environment {
        YAML_PATH = '/mnt/jenkins-setup'
    }

    stages {
        stage('Git Clone') {
            steps {
                echo 'Cloning the Git repository...'
                git 'https://github.com/ashokitschool/maven-web-app.git'
            }
        }
        stage('Maven Clean and Package') {
            steps {
                echo 'Executing Maven clean and package...'
                sh 'mvn clean package'
            }
        }
        stage('Docker Image Creation') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t anubhav .'
            }
        }
        stage('Set Kubernetes Context') {
            steps {
                script {
                    def contextExists = sh(script: 'kubectl config get-contexts | grep docker-desktop', returnStatus: true) == 0
                    if (contextExists) {
                        sh 'kubectl config use-context docker-desktop'
                    } else {
                        echo "Context docker-desktop not found"
                    }
                }
            }
        }
        stage('Kubernetes Deployment') {
            steps {
                echo "YAML Path: ${YAML_PATH}"
                sh '''
                    echo "Navigating to YAML path..."
                    cd "${YAML_PATH}"
                    pwd
                    ls -l
                    echo 'Executing Kubernetes deployment...'
                    kubectl apply -f javawebapp.yml
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed! Please check the logs for errors.'
        }
    }
}
```

---

## **Troubleshooting**

### Common Issues and Fixes

#### **1. YAML Syntax Error**
If you encounter an error like:
```
found character that cannot start any token
```
- Check the `javawebapp.yml` file for syntax issues.
- Ensure proper indentation (use spaces, not tabs).
- Use a YAML linter:
  ```bash
  yamllint javawebapp.yml
  ```

#### **2. Kubernetes Context Not Found**
If the `docker-desktop` context is missing:
- Verify the Kubernetes cluster configuration using:
  ```bash
  kubectl config get-contexts
  ```
- Switch to an available context:
  ```bash
  kubectl config use-context <your-context-name>
  ```

#### **3. Permission Issues**
If you face permission errors for the `YAML_PATH` directory:
- Update the directory permissions:
  ```bash
  chmod -R 755 /mnt/jenkins-setup
  ```

---

## **Kubernetes Deployment Example**

Here’s an example `javawebapp.yml` file for Kubernetes deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: javawebapp-deployment
  labels:
    app: javawebapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: javawebapp
  template:
    metadata:
      labels:
        app: javawebapp
    spec:
      containers:
      - name: javawebapp
        image: anubhav:latest
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: javawebapp-service
spec:
  selector:
    app: javawebapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: NodePort
```

---

## **Key Notes**
- Update the `YAML_PATH` variable to match your Jenkins file system.
- Ensure Docker, Kubernetes CLI (`kubectl`), and Maven are installed and configured on the Jenkins server.
- Verify that the Kubernetes cluster is running and accessible.

---
