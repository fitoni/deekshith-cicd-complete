pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
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

         stage("docker build and docker push"){
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
    }

}
