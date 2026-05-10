pipeline {
     agent {
        node {
            label 'roboshop' 
        } 
    }
    environment {
        appVersion = ""
        ACC_ID = "166557488034"
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
                script {
                    sh """
                        npm install
                    """
                }
            }
        }
        stage('Build image') { 
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        sh """
                            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                            docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/roboshop/catalogue:${appVersion} .
                        """
                    }  
                }
            }
        }
       stage('Push image') {
            steps {
                script {
                    docker push 166557488034.dkr.ecr.us-east-1.amazonaws.com/roboshop/catalogue:${appVersion}
                }
            }
        }     
    }
}