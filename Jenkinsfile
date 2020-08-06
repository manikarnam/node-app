currentBuild.displayName="node-app-#"+currentBuild.number
pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "sudo docker build . -t maniengg/nodeapp:${DOCKER_TAG} "
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId:'docker-hub1',variable:'dockerHubPwd')]) {
                    sh "sudo docker login -u maniengg -p ${dockerHubPwd}"
                    sh "sudo docker push maniengg/nodeapp:${DOCKER_TAG}"
                }
            }
        }
         stage('Deploy to k8s'){
	       steps{
		      sh "chmod +x changeTag.sh"
			  sh "./changeTag.sh ${DOCKER_TAG}"
			  sshagent(['kops-machine']){
			  sh "sudo scp -o StrictHostKeyChecking=no service.yml node-app-pod.yml cloud_user@mss2k8master1.eastus.cloudapp.azure.com:/home/cloud_user"
			  script{
			   try{
			       sh "ssh cloud_user@mss2k8master1.eastus.cloudapp.azure.com kubectl apply -f ."
			   }catch (error){
		                sh "ssh cloud_user@mss2k8master1.eastus.cloudapp.azure.com kubectl create -f ."
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
