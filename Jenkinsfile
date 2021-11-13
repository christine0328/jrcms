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
                         smokeTest('test')
                 }
                }
            }
        }



        
        
    }
  }

  def smokeTest(environment) {
    environment{
        CHECK_URL = "http://jrcms-${enviroment}.eba-aw7nmmrz.us-east-2.elasticbeanstalk.com/"
        CMD = "curl --write-out %{http_code} --silent --output /dev/null ${CHECK_URL}"

    }
      container('eb') {
           script{
                    sh "${CMD} > commandResult"
                    env.status = readFile('commandResult').trim()
                
                 if(env.status != 200 && env.status != 201) {
              error("Returned status code = ${env.status} when calling ${CHECK_URL}")
          }
          }
      }
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