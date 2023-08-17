pipeline {
    agent any 
    environment {
        Ansible_Server_IP = "3.85.45.201"
        Kubernetes_Server_IP = "148.100.77.142"
        DOCKERHUB_ACCESS_KEY = credentials('DOCKERHUB_ACCESS_KEY')
        DOCKERHUB_USERNAME = credentials('DOCKERHUB_USERNAME') 
    }
    stages { 
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/AkhilManoj03/JenkinsDeployKubernetes'
            }
        }
        stage("Send files to Ansible server"){
            steps{
                sshagent(['ansible_demo']) {
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@$Ansible_Server_IP'
                    sh 'scp /var/lib/jenkins/workspace/pipeline-demo/* ubuntu@$Ansible_Server_IP:/home/ubuntu'
                }
            }
        }
        stage("Build/Tag Docker image"){
            steps{
                sshagent(['ansible_demo']) {
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@$Ansible_Server_IP cd /home/ubuntu'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@$Ansible_Server_IP docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@$Ansible_Server_IP docker image tag $JOB_NAME:v1.$BUILD_ID $DOCKERHUB_USERNAME/$JOB_NAME:v1.$BUILD_ID'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@$Ansible_Server_IP docker image tag $JOB_NAME:v1.$BUILD_ID $DOCKERHUB_USERNAME/$JOB_NAME:latest'
                    
                }
            }
        }
        stage('push images to DockerHub') {
            steps{
                sshagent(['ansible_demo']) {
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@$Ansible_Server_IP docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_ACCESS_KEY'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@$Ansible_Server_IP cd /home/ubuntu'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@$Ansible_Server_IP docker push $DOCKERHUB_USERNAME/$JOB_NAME:v1.$BUILD_ID'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@$Ansible_Server_IP docker push $DOCKERHUB_USERNAME/$JOB_NAME:latest'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@$Ansible_Server_IP docker image rm $DOCKERHUB_USERNAME/$JOB_NAME:v1.$BUILD_ID $DOCKERHUB_USERNAME/$JOB_NAME:latest $JOB_NAME:v1.$BUILD_ID'
                }
            }
        }
        stage('copy files to Kubernetes Server'){
            steps{
                sshagent(['kubernetes_server']) {
                    sh 'ssh -o StrictHostKeyChecking=no linux1@$Kubernetes_Server_IP'
                    sh 'scp /var/lib/jenkins/workspace/pipeline-demo/* linux1@$Kubernetes_Server_IP:/home/linux1'
                }
            }
        }
        stage('Kubernetes Deployement using Ansible'){
            steps{
                sshagent(['ansible_demo']) {
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@$Ansible_Server_IP cd /home/ubuntu'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@$Ansible_Server_IP ansible -m ping node -u linux1 --private-key=kubkey.pem'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@$Ansible_Server_IP ansible-playbook ansible.yml -u linux1 --private-key=kubkey.pem'
                }
            }
        }
    }
}