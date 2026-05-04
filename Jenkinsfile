pipeline {
     agent {
        node {
            label 'roboshop' 
        } 
    }
    environment {
        appVersion = ""
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

        stage('Build') { 
            steps {
                echo "Building"
            }
        }
        stage('Test') { 
        steps {
            echo "Testing"
         }
       }
        stage('Deploy') { 
        steps {
            echo "Deploying"
         }
       }
    }
}