pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t sreenagandla/nodeapp:${DOCKER_TAG} "
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'sreenagandla', variable: 'Welcome@0549')]) {
                    sh "docker login -u sreenagandla -p ${dockerHubPwd}"
                    sh "docker push sreenagandla/nodeapp:${DOCKER_TAG}"
                }
            }
        }
        stage('Deploy to k8s'){
            steps{
                sh "chmod +x changeTag.sh"
                sh "./changeTag.sh ${DOCKER_TAG}"
                sshagent(['kops-machine']) {
                    sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml ubuntu@ec2-13-234-67-2:/home/ubuntu/"
                    script{
                        try{
                            sh "ssh ubuntu@ec2-13-234-67-2 kubectl apply -f ."
                        }catch(error){
                            sh "ssh ubuntu@ec2-13-234-67-2 kubectl create -f ."
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
