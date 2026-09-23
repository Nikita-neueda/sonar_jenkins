pipeline {
    agent any
    tools {
        // Automatically provisions JDK 21 and Maven 3 for the host execution environment
        jdk 'JDK21'
        maven 'Maven3'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                // -B forces batch mode (reduces noisy download logs in Jenkins)
                sh 'mvn -B clean package -DskipTests'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn -B test'
            }
            post {
                always {
                    // Captures and displays your JUnit test results on the Jenkins UI
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                // Using withCredentials guarantees the token isn't overridden by Jenkins system settings
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        mvn -B sonar:sonar \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.token=$SONAR_TOKEN \
                        -Dsonar.qualitygate.wait=true
                    '''
                }
            }
        }
        
        stage('Security Scans') {
            // Both scans run in parallel to optimize build execution time
            parallel {
                stage('Dependency Check') {
                    steps {
                        sh 'mvn -B dependency:tree'
                    }
                }
                stage('Secret Scan') {
                    steps {
                        // Scans git history for exposed API keys, passwords, or tokens
                        sh 'docker run --rm -v $(pwd):/repo -w /repo zricethezav/gitleaks:latest detect'
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo "Pipeline finished successfully! View SonarQube dashboard at http://localhost:9000"
        }
        failure {
            echo "Pipeline failed. Check the stage logs above to diagnose the error."
        }
    }
}
