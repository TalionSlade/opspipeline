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
                               
            }
        }
    }
    post{
        always{
            echo "SUCCESS"
        }
    }
}