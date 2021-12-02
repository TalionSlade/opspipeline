pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("Sonar quality check"){
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                echo "SUCCESS"
                //script{
                    // withSonarQubeEnv(credentialsId: 'sonar-token2') {
                    //    sh 'chmod +x gradlew'
                    //    sh './gradlew sonarqube --status'
                    // }
                    // timeout(time: 1, unit: 'HOURS') {
                    //   def qg = waitForQualityGate()
                    //   if (qg.status != 'OK') {
                    //       error "Pipeline aborted due to quality gate failure: ${qg.status}"
                    //   }
                    // }

                //}            
            }
        }
        stage("docker build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_password')]) {
                        sh '''
                            docker build -t 45.79.124.199:8083/springapp:${VERSION} .
                            docker login -u admin -p $nexus_password 45.79.124.199:8083
                            docker push 45.79.124.199:8083/springapp:${VERSION}
                        '''
                    }               
                }
            }
        }
    }
    post{
        post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "deekshith.snsep@gmail.com";  
		}
	}

    }
}