pipeline {
    agent any

tools{
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
       /* stage('Set kubectl context') {
    steps {
        sh 'kubectl config use-context docker-desktop'
    }
}*/
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
