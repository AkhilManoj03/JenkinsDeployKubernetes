# JenkinsDeployKubernetes
A Jenkins pipeline is initiated by a GitHub webhook. The pipeline then establishes an SSH connection to an Ansible server. This Ansible server is responsible for constructing a Docker image and subsequently uploading it to Dockerhub. Additionally, the pipeline establishes an SSH connection to a Kubernetes server. This Kubernetes server is in charge of retrieving the Docker image and creating a container within its Kubernetes cluster to run the docker image. 

