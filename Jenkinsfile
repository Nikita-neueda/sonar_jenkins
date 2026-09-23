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
        withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
            sh '''
                # 1. Clear the local scanner cache to reset the project state
                rm -rf /var/lib/jenkins/.sonar/cache
                
                # 2. Use 'verify' so Jenkins actually executes the tests and generates coverage reports
                mvn -B clean verify sonar:sonar \
                -Dsonar.host.url=http://localhost:9000 \
                -Dsonar.token=$SONAR_TOKEN \
                -Dsonar.qualitygate.wait=false
            '''
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
