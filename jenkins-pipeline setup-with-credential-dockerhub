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

        /* stage('Set kubectl context') {
            steps {
                sh 'kubectl config use-context docker-desktop'
            }
        }*/

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
