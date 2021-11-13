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
        stage("docker build"){
            steps{
                 sh 'docker build -t 34.125.224.169:8084/springapp:${VERSION} .'
                 }
             }
        stage("docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus-password-credentials', variable: 'docker_password_generated')]) {
                             sh '''
                                docker login -u admin -p $docker_password_generated 34.125.224.169:8084
                                docker push  34.125.224.169:8084/springapp:${VERSION}
                                docker rmi 34.125.224.169:8084/springapp:${VERSION}
                            '''
                    }
                }
            }
        }
        stage("pushing the helm charts to nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus-password-credentials', variable: 'docker_password_generated')]) {
                          dir('kubernetes/') {
                             sh '''
                                 helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                 tar -czvf  myapp-${helmversion}.tgz myapp/
                                 curl -u admin:$docker_password_generated http://34.125.224.169:8081/repository/helm-package/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                          }
                    }
                }
            }
        }
        stage('Deploying application on k8s cluster') {
            steps {
               script{
                   withCredentials([kubeconfigFile(credentialsId: 'configConfig-files', variable: 'KUBECONFIG')]) {
                        dir('kubernetes/') {
                          sh 'helm upgrade --install --set image.repository="34.125.224.169:8084/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ '
                        }
                    }
               }
            }
        }
        stage('verifying app deployment'){
            steps{
                script{
                     withCredentials([kubeconfigFile(credentialsId: 'configConfig-files', variable: 'KUBECONFIG')]) {
                         sh 'kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl myjavaapp-myapp:8080'

                     }
                }
            }
        }
    }

    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "deekshith.snsep@gmail.com";
		 }
	 }
}
