pipeline{
    agent any
    environment{
          VERSION = "${env.BUILD_ID}"
        }
    docker {
          image 'openjdk:11'
            args '-v "$PWD":/app'
            reuseNode true
         }
  stages{
        stage ('Test & Build Artifact') {
              agent {
                steps {
                    sh 'chmod +x gradlew'
                     sh './gradlew clean build'
                       }
                    }
                 }
        stage("docker build"){
            steps{
                 sh 'docker build -t 34.125.127.76:8085/gradle-hosted-8085:${VERSION} .'
                 }
             }
        stage("docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_docker_password_generated', variable: 'nexus_docker_password_generated')]) {
                             sh '''
                                docker login -u admin -p $nexus_docker_password_generated 34.125.127.76:8085
                                docker push  34.125.127.76:8085/gradle-hosted-8085:${VERSION}
                                docker rmi 34.125.127.76:8085/gradle-hosted-8085:${VERSION}
                            '''
                    }
                }
            }
        }
        stage("pushing the helm charts to nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_docker_password_generated', variable: 'nexus_docker_password_generated')]) {
                          dir('kubernetes/') {
                             sh '''
                                 helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                 tar -czvf  myapp-${helmversion}.tgz myapp/
                                 curl -u admin:$nexus_docker_password_generated http://34.125.127.76:8081/repository/gradle-helm-package/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                          }
                    }
                }
            }
        }
        stage('Deploying application on k8s cluster') {
            steps {
               script{
                  withCredentials([kubeconfigFile(credentialsId: 'kubernetes-configuration', variable: 'KUBECONFIG')])   {
                        dir('kubernetes/') {
                          sh 'helm upgrade --install --set image.repository="34.125.127.76:8085/gradle-hosted-8085" --set image.tag="${VERSION}" myjavaapp myapp/ '
                        }
                    }
               }
            }
        }
        stage('verifying app deployment'){
            steps{
                script{
                     withCredentials([kubeconfigFile(credentialsId: 'kubernetes-configuration', variable: 'KUBECONFIG')])  {
                         sh 'kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl myjavaapp-myapp:8080'

                     }
                }
            }
        }

    }
}
