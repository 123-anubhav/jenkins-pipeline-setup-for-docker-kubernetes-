## working ci/cd pipeline for aws only docker used for image creation and image run

---
pipeline {
    agent any

tools{
    maven 'mavn'
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
        stage('docker container'){
            steps{
                sh 'docker run -d -p 9090:8080 --name anubhav_container anubhav:latest'
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

---

pipeline {
    agent any

tools{
    maven 'mavn'
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
        stage('docker container'){
            steps{
                sh 'docker run -d -p 9090:8080 --name anubhav_container anubhav:latest'
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
