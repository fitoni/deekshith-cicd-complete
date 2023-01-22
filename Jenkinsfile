pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            agent {
                docker {
                    image 'openjdk:8'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-update-token') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube'
                    }

                }  
            }
        }
              
    }

}
