# Jenkins Pipeline Script Documentation

This documentation outlines a Jenkins pipeline script that automates the build, Docker image creation, and Kubernetes deployment of a Maven-based web application.

---

## **Pipeline Overview**
The pipeline consists of the following stages:
1. **Git Clone**
2. **Maven Clean and Package**
3. **Docker Image Creation**
4. **Set `kubectl` Context**
5. **Kubernetes Deployment**

---

### **Pipeline Script**

```groovy
pipeline {
    agent any

    tools {
        maven 'm3'
    }

    environment {
        YAML_PATH = '/mnt/jenkins-setup' // Path to Kubernetes YAML configuration files
    }

    stages {
        stage('Git Clone') {
            steps {
                echo 'Cloning the Git repository'
                git 'https://github.com/ashokitschool/maven-web-app.git'
            }
        }
        
        stage('Maven Clean and Package') {
            steps {
                echo 'Executing Maven clean and package'
                sh 'mvn clean package'
            }
        }
        
        stage('Docker Image Creation') {
            steps {
                echo 'Building Docker image'
                sh 'docker build -t anubhav .'
            }
        }
        
        stage('Set kubectl Context') {
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
        
        stage('Kubernetes Deployments') {
            steps {
                echo "YAML Path: ${YAML_PATH}"
                sh '''
                    echo "Navigating to YAML path"
                    cd "${YAML_PATH}"
                    pwd
                    ls -l
                    echo 'Executing Kubernetes deployments'
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

### **Pipeline Breakdown**

#### **1. Git Clone**
- Clones the source code from the specified GitHub repository.
- Command: 
  ```groovy
  git 'https://github.com/ashokitschool/maven-web-app.git'
  ```

#### **2. Maven Clean and Package**
- Runs Maven commands to clean the workspace and build the application.
- Command:
  ```bash
  mvn clean package
  ```

#### **3. Docker Image Creation**
- Builds a Docker image using the `Dockerfile` in the repository.
- Command:
  ```bash
  docker build -t anubhav .
  ```

#### **4. Set `kubectl` Context**
- Verifies if the Kubernetes context `docker-desktop` exists and sets it as the active context.
- Command:
  ```bash
  kubectl config use-context docker-desktop
  ```

#### **5. Kubernetes Deployments**
- Navigates to the specified YAML path and applies the Kubernetes deployment configuration.
- Commands:
  ```bash
  cd "${YAML_PATH}"
  kubectl apply -f javawebapp.yml
  ```

---

### **Post Actions**

- **Success:** Prints a success message upon pipeline completion.
- **Failure:** Prints an error message if any stage fails.

---

### **Key Points**
- The `YAML_PATH` is used to store Kubernetes deployment YAML files. Update it according to your setup.
- The pipeline handles dynamic context detection for `kubectl`.
- Ensure the `m3` Maven installation is configured in Jenkins.

---

### **Requirements**
1. Jenkins with the following plugins:
   - **Pipeline**
   - **Git**
2. Pre-installed tools on the Jenkins server:
   - Docker
   - Kubernetes CLI (`kubectl`)
   - Maven (`m3` configured in Jenkins)
3. Proper Kubernetes cluster setup with the required context (`docker-desktop` in this example).
4. Kubernetes YAML deployment file (`javawebapp.yml`) placed in the `YAML_PATH` directory.

---
