Certainly! Here's a detailed to explain your Jenkins declarative pipeline, 
including the step-by-step guide for using credentials in Jenkins:

---

# Jenkins Pipeline Setup for Docker and Kubernetes

Jenkins setup using Docker Desktop, including Docker and Kubernetes installation inside Jenkins, and a Jenkins pipeline setup for building, pushing Docker images, and deploying to Kubernetes.

## Pipeline Stages

### Step 1: Pipeline Configuration

```groovy
pipeline {
    agent any

    tools {
        maven 'm3'
    }

    environment {
        YAML_PATH = '/mnt/jenkins-setup'
        DOCKER_EMAIL = 'anubhav.aa2@gmail.com'
    }

    stages {
        stage('git clone') {
            steps {
                echo 'git clone'
                git 'https://github.com/ashokitschool/maven-web-app.git'
            }
        }

        stage('maven clean package') {
            steps {
                echo 'maven clean package'
                sh 'mvn clean package'
            }
        }

        stage('docker image creation and Push') {
            steps {
                echo 'Building and pushing Docker image...'
                // Retrieve Docker credentials and use them
                withCredentials([usernamePassword(credentialsId: '5abnd5f-50ee-4iod6-a5a9-851c48hj54f', 
                                usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                    docker build -t ${DOCKER_USERNAME}/anubhav:latest .
                    docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}
                    docker push ${DOCKER_USERNAME}/anubhav:latest
                    '''
                }
            }
        }

        stage('Create Kubernetes Secret') {
            steps {
                echo 'Creating Kubernetes secret...'
                // Retrieve Docker credentials for Kubernetes secret creation
                withCredentials([usernamePassword(credentialsId: '5abnd5f-50ee-4iod6-a5a9-851c48hj54f', 
                                usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                    kubectl create secret docker-registry my-registry-key \
                    --docker-username=${DOCKER_USERNAME} \
                    --docker-password=${DOCKER_PASSWORD} \
                    --docker-email=${DOCKER_EMAIL} || echo "Secret already exists"
                    '''
                }
            }
        }

        stage('Set kubectl context') {
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

        stage('ks deployments') {
            steps {
                echo "YAML Path: ${YAML_PATH}"
                sh 'kubectl config use-context docker-desktop || echo "Context docker-desktop not found"'
                sh '''
                    echo "Current Directory:"
                    cd "${YAML_PATH}"
                    pwd
                    ls -l
                    echo 'kubernetes deployments execution starts!............!...........!...........!'
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

### Explanation of Each Stage

1. **Pipeline Configuration**:
   - **agent any**: The pipeline can run on any available Jenkins agent.
   - **tools**: Specifies the Maven version to use (`m3`).
   - **environment**: Defines environment variables `YAML_PATH` and `DOCKER_EMAIL`.

2. **Stages**:
   - **git clone**: 
     - Clones the Git repository containing the Maven web application.
     - `git 'https://github.com/ashokitschool/maven-web-app.git'` fetches the latest code.
   - **maven clean package**: 
     - Runs `mvn clean package` to clean and package the Maven project.
   - **docker image creation and Push**: 
     - Builds a Docker image named `anubhav` from the Dockerfile in the project directory.
     - Pushes the Docker image to Docker Hub using credentials securely retrieved from Jenkins.
   - **Create Kubernetes Secret**: 
     - Creates a Kubernetes secret for Docker registry credentials using credentials securely retrieved from Jenkins.
   - **Set kubectl context**: 
     - Checks if the `docker-desktop` context exists and sets it as the current context for `kubectl`.
     - If the context is not found, it logs an appropriate message.
   - **ks deployments**:
     - Uses the `kubectl` command to apply Kubernetes configurations from the specified YAML file.
     - Displays the current directory and lists its contents before applying the configurations.
     - Deploys the `javawebapp.yml` Kubernetes configuration.

3. **Post Actions**:
   - **success**: Prints a success message upon successful pipeline completion.
   - **failure**: Prints a failure message if the pipeline encounters any errors.

## Step-by-Step Guide to Use Credentials in Jenkins

### 1. Add Credentials to Jenkins
1. Go to your Jenkins dashboard.
2. Navigate to **Manage Jenkins** → **Manage Credentials**.
3. Select the appropriate scope (e.g., Global or specific folder/job scope).
4. Click **Add Credentials**.
5. Fill in the following details:
   - **Kind**: Username with password.
   - **Username**: Your Docker Hub username.
   - **Password**: Your Docker Hub password.
   - **ID**: Set an identifier like `docker-username-password`.
   - **Description**: Optionally provide a description.

### 2. Access Credentials in Pipeline Script
Use the `withCredentials` block in your Jenkins pipeline to retrieve the credentials securely.

```groovy
withCredentials([usernamePassword(credentialsId: '5a0e2d5f-00ee-4ed6-a5a9-851c48af554f', 
                usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
    // Your script here
}
```

### Additional Troubleshooting

If you face any file permission issues inside the Jenkins container, execute the following command:

```bash
chmod -R 755 /mnt/jenkins-setup
```

### Conclusion

This pipeline automates the process of cloning the repository, building the Maven project, creating and pushing a Docker image, creating a Kubernetes secret, setting the Kubernetes context, and deploying the application. It ensures a streamlined CI/CD workflow for your Maven-based web application using Jenkins, Docker, and Kubernetes.

---
