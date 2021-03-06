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
                            tar -czvf myapp-0.2.0.tgz kubernetes/myapp/
                            curl -u admin:$nexus_password http://45.79.124.199:8081/repository/helm-repo/ --upload-file myapp-0.2.0.tgz -v
                        '''
                    }               
                }
            }
        }        
        stage('Deploying application to K8S cluster') {
            steps {
                script{
                    withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
                        dir('kubernetes/'){
                            sh 'helm upgrade --install --set image.repository="45.79.124.199:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
                        }
                    }
                }
            }
        }
        stage('Verifying app deployment') {
            steps {
                script{
                    withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
                        sh 'kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl myjavaapp-myapp:8080'
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