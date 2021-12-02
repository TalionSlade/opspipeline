pipeline{
    agent any
    stages{
        stage("Sonar quality check"){
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                echo "SUCCESS"
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token2') {
                       sh 'chmod +x gradlew'
                       sh './gradlew sonarqube'
                    }
                    // timeout(time: 1, unit: 'HOURS') {
                    //   def qg = waitForQualityGate()
                    //   if (qg.status != 'OK') {
                    //        error "Pipeline aborted due to quality gate failure: ${qg.status}"
                    //   }
                    // }

                }            
            }
        }
    }
    post{
        always{
            echo "SUCCESS"
        }
    }
}