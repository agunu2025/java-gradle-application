pipeline {
    agent any

    environment {
        REMOTE_ADDRESS = "REPLACE_WITH_REMOTE_ADDRESS"
    }

    stages {
        stage ('Test & Build Artifact') {
            agent {
                docker {
                    image 'openjdk:11'
                    args '-v "$PWD":/app'
                    reuseNode true
                }
            }
            steps {
                sh './gradlew clean build'
            }
        }
        
    }
}
