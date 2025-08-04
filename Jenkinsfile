pipeline {
    agent any

    environment {
        MAVEN_HOME = tool 'Maven3'
        PATH = "${env.MAVEN_HOME}/bin:${env.PATH}"
    }

    stages {
        stage('Build & Package') {
            steps {
                echo "Checking out source code on agent: ${env.NODE_NAME}"
                checkout scm

                echo "Compiling and packaging the project..."
                sh 'mvn clean package'

                echo "Stashing the JAR artifact..."
                stash name: 'jar-artifact', includes: 'rest-api/target/*.jar'

                echo "Stashing the full workspace for the test stage..."
                stash name: 'workspace-for-testing', includes: '**/**'
            }
        }

        stage('Test') {
            steps {
                echo "Unstashing workspace on agent: ${env.NODE_NAME}"
                unstash 'workspace-for-testing'

                echo "Running tests..."
                sh 'mvn test'
            }
        }

        stage('Archive Artifact') {
            steps {
                echo "Unstashing the JAR artifact on agent: ${env.NODE_NAME}"
                unstash 'jar-artifact'

                echo "Archiving the JAR file..."
                archiveArtifacts artifacts: 'rest-api/target/*.jar', fingerprint: true
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished. Cleaning up workspace.'
            cleanWs()
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
