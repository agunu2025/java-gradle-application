pipeline {
    agent any

    triggers{
        bitbucketPush()
    }

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
        stage ('Build & Push docker image') {
            steps {
                withDockerRegistry(credentialsId: '9b38192f-91c0-4789-ac24-85baabb4e094', url: 'https://index.docker.io/v1/') {
                    sh 'docker push turkogluc/spring-jenkins-demo'
                }
            }
        }
        stage ('Deploy image to remote server') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'instance-1', keyFileVariable: 'KEY_FILE', usernameVariable: 'USER')]) {
                    sh 'ssh -i ${KEY_FILE} ${USER}@${REMOTE_ADDRESS} "docker run -d turkogluc/spring-jenkins-demo"'
                }
            }
        }
    }
}
