---

# Jenkins Declarative Pipeline for Docker and Kubernetes

This Jenkins pipeline is designed to automate the building, testing, and deployment of a Maven-based web application using Docker and Kubernetes.

## Pipeline Stages

### Pipeline Configuration

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
        stage('docker image creation') {
            steps {
                echo 'docker image creation'
                sh 'docker build -t anubhav .'
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
   - **environment**: Defines an environment variable `YAML_PATH` pointing to the Jenkins setup directory.

2. **Stages**:
   - **git clone**: 
     - Clones the Git repository containing the Maven web application.
     - `git 'https://github.com/ashokitschool/maven-web-app.git'` fetches the latest code.
   - **maven clean package**: 
     - Runs `mvn clean package` to clean and package the Maven project.
   - **docker image creation**: 
     - Builds a Docker image named `anubhav` from the Dockerfile in the project directory.
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

### Additional Troubleshooting

If you face any file permission issues inside the Jenkins container, execute the following command:

```bash
chmod -R 755 /mnt/jenkins-setup
```

### Conclusion

This pipeline automates the process of cloning the repository, building the Maven project, creating a Docker image, setting the Kubernetes context, and deploying the application. It ensures a streamlined CI/CD workflow for your Maven-based web application using Jenkins, Docker, and Kubernetes.

---
