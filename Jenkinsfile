podTemplate(
    containers: [
        containerTemplate(image: 'docker', name: 'docker', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'eb', image: 'mini/eb-cli', command: 'cat', ttyEnabled: true)], 
    volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]) {
    node(POD_LABEL) {
        
        stage('Check directory') {
            sh 'ls -lah'
        }
         
    
        container('docker'){
            withCredentials([usernamePassword(credentialsId: 'DockerCredential', usernameVariable: 'USER', passwordVariable: 'PASSWD')]) {
                  stage('Build') {
                    checkout scm
                    sh 'docker build -t kriscloud001/jrcms-private:V4 .'
                  }
                 stage('Docker hub login') {
                    sh 'docker login --username ${USER} --password ${PASSWD}'
                  }
                  stage('Docker push') {
                      sh 'docker push kriscloud001/jrcms-private:V4'
                  }
                 if (env.BRANCH_NAME == 'master') {
    
                    stage("Deploy to test environment") {
                        
                        deployToEB('test')
                    }
                    stage("Integration test to test environment") {  
                        def CHECK_URL = "http://jrcms-test.eba-aw7nmmrz.us-east-2.elasticbeanstalk.com/"
                        def response = sh(script: "/usr/bin/curl -sLI -w %{http_code} ${CHECK_URL} -o /dev/null", returnStdout: true)

                                sh "echo ${response}"
                                if (response == '200') {
                                    currentBuild.result = "SUCCESS"
                                }
                                else {
                                    currentBuild.result = "FAILURE"
                                }
                            
                        
                    }
                }
            }
        }



        
        
    }
  }

  def smokeTest(environment) {
            
        
  
  }
  
   
    def deployToEB(environment) {
            checkout scm
            withCredentials([usernamePassword(credentialsId: 'aws-eb', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                container('eb') {
                    withEnv(["AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}", "AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}", "AWS_REGION=us-east-2"]) {
                        dir("deployment") {
                        sh "sh generate-dockerrun.sh ${currentBuild.number}"
                        sh "eb deploy Jrcms-${environment} -l ${currentBuild.number} --region us-east-2"
                        }
                    }
                }    
            }
     }