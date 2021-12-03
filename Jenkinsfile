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
                            docker build --network=host -t 45.79.124.199:8083/springapp:${VERSION} .
                            docker login -u admin -p $nexus_password 45.79.124.199:8083
                            docker push 45.79.124.199:8083/springapp:${VERSION}
                        '''
                    }               
                }
            }
        }
        stage("identifying misconfigs using datree in helm charts"){
            steps{
                script{
                    dir('kubernetes/'){
                        withEnv(['DATREE_TOKEN =iRmdw4ZUKQMWMyXReFmVDj']){
                            sh 'helm datree test myapp/'
                        }
                    }            
                }
            }
        }
        stage("pushing helm charts to nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_password')]) {
                        sh '''
                            helmversion = $(helm show chart kubernetes/myapp/ | grep version | cut -d: -f 2 | tr -d '')
                            tar -czvf myapp-${helmversion}.tgz myapp/
                            curl -u admin:$docker_password http://45.79.124.199:8081/repository/helm-repo/ --upload-file myapp-0.2.0.tgz -v
                        '''
                    }               
                }
            }
        }
    }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "arpan01ghosh1307@gmail.com";  
		}
	}
}