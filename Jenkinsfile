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
        stage ('Unit tests') {
            steps {
                script {
                    sh """
                        npm test
                    """
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool name: "sonar-8"  //agent configuration
                    withSonarQubeEnv('sonar-server') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
        stage ('Quality Gate'){
            steps {
                timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
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
                    sh """
                        docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/roboshop/catalogue:${appVersion}
                    """
                }
            }
        }     
    }
    // post build
    post { 
        always { 
            echo 'Clean workspace in Jenkins agent'
            cleanWs()
        }
        success {
            echo "pipeline success"
        }
        failure {
            echo "pipeline failure"
        }
    }
}