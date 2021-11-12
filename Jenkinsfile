pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage ('Test & Build Artifact') {
         agent {
             docker {
                     image 'openjdk:11'
                     args '-v "$PWD":/app'
                     reuseNode true
                  }
               }
          steps {
            sh 'chmod +x gradlew'
            sh './gradlew clean build'
            }
         }

	  }
}
