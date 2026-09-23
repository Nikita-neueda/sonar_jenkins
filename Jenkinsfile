pipeline {
    agent any

    tools {
        // Ensures Maven is installed and accessible in the runtime environment
        maven 'Maven3' 
    }

    environment {
        // Securely binds the Jenkins secret text credential to an environment variable
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {
        stage('Clone Source Code') {
            steps {
                // Clones your specific codebase branch
                checkout scm
            }
        }

        stage('Compile & Test') {
            steps {
                // Compiles the codebase before scanning
                sh 'mvn clean compile'
            }
        }

        stage('SonarQube Code Analysis') {
            steps {
                // Injects the configured SonarQube server environment details automatically
                withSonarQubeEnv('SonarQube-Local') {
                    // Runs the target Maven scanner command using variables
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.token=${SONAR_TOKEN} \
                        -Dsonar.qualitygate.wait=true
                    """
                }
            }
        }
    }

    post {
        success {
            echo "SonarQube analysis finished successfully! View results at http://localhost:9000"
        }
        failure {
            echo "Pipeline or Analysis failed. Review the terminal outputs above."
        }
    }
}
