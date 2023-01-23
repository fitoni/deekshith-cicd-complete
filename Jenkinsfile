pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("Sonar Quality Check"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-update-token') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube'
                    }
                    timeout(10) {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                }  
            }
        }

         stage("Docker Build and Push onto Nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_password_in_nexus', variable: 'docker_password_in_nexus')]) {
                        sh '''
                            docker build -t 108.137.7.77:8083/springapp:${VERSION} . 
                            docker login -u admin -p $docker_password_in_nexus 108.137.7.77:8083
                            docker push 108.137.7.77:8083/springapp:${VERSION}
                            docker rmi 108.137.7.77:8083/springapp:${VERSION}
                        '''
                    }
                }
            }       
         }  

         stage("Identifying Misconfigurations using Datree"){
            steps{
                script{
                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=902b9cb1-1099-4c38-996d-89031576735c']) {
                              sh 'helm datree test myapp/'
                        }
                        
                    }
                }
            }
         }   

         stage("Push Helm Charts onto Nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_password_in_nexus', variable: 'docker_password_in_nexus')]) {
                        dir('kubernetes/'){
                            sh '''
                                helmversion=$(helm show chart myapp/ | grep version | cut -d: -f 2 | tr -d ' ') 
                                tar -czvf myapp-${helmversion}.tgz myapp/
                                curl -u admin:$docker_password_in_nexus http://108.137.7.77:8081/repository/deekshith-helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                            '''                            
                        }
                    }
                }
            }       
         } 

    }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "fitoni77@gmail.com";  
		}
	}
}
