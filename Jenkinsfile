pipeline {
     agent {
        node {
            label 'roboshop' 
        } 
    }
    environment {
        appVersion = ""
        ACC_ID = "166557488034"
        region = "us-east-1"
    }
    parameters {
        choice(name: 'ENVIRONMENT', 
        choices: ['DEV', 'UAT', 'PROD'], 
        description: 'Select the environment to deploy')
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
        // stage ('Unit tests') {
        //     steps {
        //         script {
        //             sh """
        //                 npm test
        //             """
        //         }
        //     }
        // }
        // stage('SonarQube Analysis') {
        //     steps {
        //         script {
        //             def scannerHome = tool name: "sonar-8"  //agent configuration
        //             withSonarQubeEnv('sonar-server') {
        //                 sh "${scannerHome}/bin/sonar-scanner"
        //             }
        //         }
        //     }
        // }
        // stage ('Quality Gate'){
        //     steps {
        //         timeout(time: 1, unit: 'HOURS') {
        //         waitForQualityGate abortPipeline: true
        //         }
        //     }
        // }
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
        // stage('Trivy OS Scan') {
        //     steps {
        //         script {
        //             // Generate table report
        //             sh """
        //                 trivy image \
        //                     --scanners vuln \
        //                     --pkg-types os \
        //                     --severity HIGH,MEDIUM \
        //                     --format table \
        //                     --output trivy-os-report.txt \
        //                     --exit-code 0 \
        //                     ${ACC_ID}.dkr.ecr.${region}.amazonaws.com/roboshop/catalogue:${appVersion}
        //             """

        //             // Print table to console
        //             sh 'cat trivy-os-report.txt'

        //             // Fail pipeline if vulnerabilities found
        //             def scanResult = sh(
        //                 script: """
        //                     trivy image \
        //                         --scanners vuln \
        //                         --pkg-types os \
        //                         --severity HIGH,MEDIUM \
        //                         --format table \
        //                         --exit-code 1 \
        //                         --quiet \
        //                         ${ACC_ID}.dkr.ecr.${region}.amazonaws.com/roboshop/catalogue:${appVersion}
        //                 """,
        //                 returnStatus: true
        //             )

        //             if (scanResult != 0) {
        //                 error "🚨 Trivy found HIGH/MEDIUM OS vulnerabilities. Pipeline failed."
        //             } else {
        //                 echo "✅ No HIGH or MEDIUM OS vulnerabilities found. Pipeline continues."
        //             }
        //         }
        //     }
        // }
        // stage('Trivy Dockerfile Scan') {
        //     steps {
        //         script {
        //             sh """
        //                 trivy config \
        //                     --severity HIGH,MEDIUM \
        //                     --format table \
        //                     --output trivy-dockerfile-report.txt \
        //                     Dockerfile
        //             """

        //             sh 'cat trivy-dockerfile-report.txt'

        //             def scanResult = sh(
        //                 script: """
        //                     trivy config \
        //                         --severity HIGH,MEDIUM \
        //                         --exit-code 1 \
        //                         --format table \
        //                         Dockerfile
        //                 """,
        //                 returnStatus: true
        //             )

        //             if (scanResult != 0) {
        //                 error "🚨 Trivy found HIGH/MEDIUM misconfigurations in Dockerfile. Pipeline failed."
        //             } else {
        //                 echo "✅ No HIGH or MEDIUM Dockerfile misconfigurations found. Pipeline continues."
        //             }
        //         }
        //     }
        // }
        stage('Push image') {
            steps {
                script {
                    sh """
                        docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/roboshop/catalogue:${appVersion} 
                    """
                }
            }
        }
        stage('Checkout catalogue manifests') {    
            steps {
                dir('k8s-ingress') {      
                    git url: 'https://github.com/ImManiKanta/k8s-ingress.git',
                        branch: 'main'
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    if (params.ENVIRONMENT == 'PROD') {
                         echo "Deploying to PRODUCTION"
                    } 
                    else if (params.ENVIRONMENT == 'UAT') {
                        echo "Deploying to UAT"
                        withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        sh """
                            aws eks update-kubeconfig --region us-east-1 --name roboshop 
                            kubectl create namespace uat
                            kubectl apply -f  k8s-ingress/backend/manifest.yaml -n uat
                        """
                        }   
                    } 
                    else {
                        echo "Deploying to Dev"
                        withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        sh """
                            aws eks update-kubeconfig --region us-east-1 --name roboshop 
                            sed -i 's/1.0.0/${appVersion}/g'  k8s-ingress/backend/manifest.yaml
                            cd k8s-ingress/
                            kubectl apply -f namespace.yaml
                            kubectl apply -f backend/manifest.yaml -n roboshop
                        """
                        }
                    }
                }
            }
        }      
    }
    // post build
    post { 
        always { 
            echo 'Clean workspace in Jenkins agent'
            //cleanWs()
        }
        success {
            echo "pipeline success"
        }
        failure {
            echo "pipeline failure"
        }
    }
}