pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t maniengg/nodeapp:${DOCKER_TAG} "
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId:'docker-hub1',variable:'dockerHubPwd')]) {
                    sh "docker login -u maniengg -p ${dockerHubPwd}"
                    sh "docker push maniengg/nodeapp:${DOCKER_TAG}"
                }
            }
        }
        stage('Deploy to k8s'){
          steps{
              sh "chmod +x changeTag.sh"
               sh "./changeTag.sh ${DOCKER_TAG}"
                sshagent(['Centos-machine']) {
                  sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml mkarnam@192.168.228.143:/home/mkarnam/"
                   script{
                       try{
                         sh "ssh mkarnam@192.168.228.143 kubectl apply -f ."
                          }catch(error){
                            sh "ssh mkarnam@192.168.228.143 kubectl create -f ."
                        }
                    }
                }
             }
        }
    }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
