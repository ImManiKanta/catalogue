pipeline {
     agent {
        node {
            label 'roboshop' 
        } 
    }
    environment {
        appVersion = ""
    }
    options {
        disableConcurrentBuilds()
    }
    stages {     
        stage('Read version'){
            steps {
                script {
                    // Load and parse the JSON file
                    def packageJson = readJSON file: 'package.json'
                    
                    // Access fields directly
                    appVersion = packageJson.version
                    echo "Building version ${appVersion}"
                }
            }
        }

        stage('Install dependencies') { 
            steps {
                script{
                    sh """
                        npm install
                    """
                }
            }
        }
        stage('Build image') { 
            steps {
                script {
                    sh """
                    docker build -t catalogue:${appVersion} .
                """
                }  
            }
       }
        
    }
}