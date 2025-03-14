pipeline {
    
    agent {
        label "master"
    }

    environment {
       DOCKER_USER='80963'
       DOCKER_PASSWORD='S@tya954'
       KUBE_NAMESPACE='satya-test-kube'
    }
    
    stages {
        stage("Checkout From Git") {
            steps {
                git branch: 'main', url: 'https://github.com/satya954/deploy-flaskApp-cicd'
            }
        }
        
        stage("Building Docker Image") {
            steps {
                script {
                    sh "docker build -t ${DOCKER_USER}/flask-app:${BUILD_NUMBER} ."
                }
            }
        }
        
        stage("Testing Environnment"){
            steps{
                sh "docker run -dit --name TestingApp-${BUILD_NUMBER} -p ${BUILD_NUMBER}:8080 ${DOCKER_USER}/flask-app:${BUILD_NUMBER}"
                retry(30){
                    sh "curl http://192.168.1.50:${BUILD_NUMBER}/ "
                }
            }
        }
        
        stage("Testing"){
            steps{
                retry(30){
                    sh "curl http://192.168.1.50:${BUILD_NUMBER}/ "
                }
            }
        }
        
        stage("Approving"){
            steps{
                script{
                    Boolean userInput = input(id: 'Proceed1', message: 'Ready To Go?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']])
                    echo 'userInput: ' + userInput

                    if(userInput == true){
                        // do action
                    } else {
                        // not do action
                        echo "Action was aborted."
                    }
                }
            }
        }
        
        stage ("Pushing To Dockerhub"){
            steps{
                withCredentials([string(credentialsId: 'docker-credentials-50', variable: 'DOCKER_PASSWORD')]){
                    sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASSWORD} "
                    sh "docker push ${DOCKER_USER}/flask-app:${BUILD_NUMBER}"
                }
            }
        }
        
        stage("Deploying On Kubernetes"){
            steps{
                sshagent(["135_root_user"]){
                    sh "ssh -o StrictHostKeyChecking=no root@192.168.1.235 rm -rvf /root/deploy-flaskApp-cicd/ "
                    sh "ssh root@192.168.1.135 git clone https://github.com/satya954/deploy-flaskApp-cicd/"
                    sh "ssh root@192.168.1.235 sed -i s/replaceImageTag/${BUILD_NUMBER}/g /root/deploy-flaskApp-cicd/deployment.yaml"
                    sh "ssh root@192.168.1.235 kubectl -n ${KUBE_NAMESPACE} apply -f /root/deploy-flaskApp-cicd/deployment.yaml"
                    sh "ssh root@192.168.1.235 kubectl -n ${KUBE_NAMESPACE} apply -f /root/deploy-flaskApp-cicd/service.yaml"
                }   
            }        
        }
    }
}
