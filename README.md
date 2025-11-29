# CI/CD pipeline to deploy MEAN stack app
![dd_assignment_1](https://github.com/user-attachments/assets/0a8afeb6-6ccc-4970-a8a3-0453eb5119e4)

## Project Overview

A containerized full-stack MEAN (MongoDB, Express, Angular, Node.js) application integrated with a Jenkins CI/CD pipeline. It automates the process of building, testing, and deploying using Docker, Docker Compose, and GitHub Webhooks, with Docker Hub for image storage and Nginx for frontend delivery and reverse proxying.  

### Key Files

- [backend/Dockerfile](backend/Dockerfile) – Docker setup for the Node.js backend
- [frontend/Dockerfile](frontend/Dockerfile) – Docker setup for the Nginx-Angular frontend
- [frontend/nginx.conf](frontend/nginx.conf) – Custom Nginx config for frontend deployment
- [Jenkinsfile](./Jenkinsfile) - Contains jenkins pipeline script
- [docker-compose.yml](./docker-compose.yml) - docker-compose file to run frontend, backend, database containers together as a same service
  
  
### Project Structure

<pre lang="markdown"> 
📁 MEAN-app-docker-deploy 
├── 📁 backend 
│   ├── 📁 app 
│   ├── 📄 server.js 
│   └── 📄 Dockerfile
│   
├── 📁 frontend 
│   ├── 📁 src 
|   ├── 📄 nginx.conf
│   └── 📄 Dockerfile 
│   
├── 📄 Jenkinsfile
├── 📄 docker-compose.yml
└── 📄 README.md
</pre>

## System Configurations
> [!NOTE]
> AWS ec2 Ubuntu VM , 2vCPU 8GiB of memory and 20GiB of storage
### Install git, java, docker, docker-compose and jenkins
All these installations and configurations are done using user data script which will be the first script to be run as root user after creating the VM.  
<pre>#!/bin/bash
apt update -y
apt upgrade -y
apt install git openjdk-17-jre-headless docker.io docker-compose -y
wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
apt-get update
apt-get install jenkins -y
usermod -aG docker jenkins
systemctl restart jenkins
systemctl enable jenkins</pre>


## Jenkins Configurations
To visualize and manage your CI/CD pipeline effectively, install the following recommended Jenkins plugins:

- **Pipeline: Stage View** – Visualize stages in your pipeline.

>  You can install these from **Jenkins Dashboard → Manage Jenkins → Plugins**.

### Jenkins Credentials

- Add **DockerHub credentials** (`Username with token`) to Jenkins under `Manage Jenkins → Credentials → Global` for secure authentication.
- These credentials are used in the Jenkins pipeline to **push Docker images** to DockerHub without exposing sensitive data in the script.

<br/>

![jenkins_cred](https://github.com/user-attachments/assets/8b139e6a-8762-478c-91dc-3bb20744242e)

<br/>

- Dockerhub token with read write access

<br/>

![docker_cred](https://github.com/user-attachments/assets/7c5b0b63-a6c9-48e4-8e94-56ece7502fad)


### Jenkins Pipeline

- Go to Jenkins Dashboard → New Item, choose Pipeline, and give it a meaningful name and description in the General section.

![image](https://github.com/user-attachments/assets/e85f5e32-7392-4102-998e-9bf71db79b6d)

<br/>

- In the Build Triggers section, select "GitHub hook trigger for GITScm polling" to automatically run the pipeline when a new commit is pushed.

![image](https://github.com/user-attachments/assets/bcf85ec5-1526-449a-b871-97e6d254c158)

<br/>

- In the Pipeline section, choose "Pipeline script from SCM", set SCM to Git, paste your GitHub repository URL, select the desired branch, and provide the path to your Jenkinsfile and then apply and save.

![image](https://github.com/user-attachments/assets/47b5b52b-0d49-46b3-8958-734cc7c8000e)





## GitHub Webhook Setup

- In your **repository settings**, go to **Webhooks** and add the **Payload URL** and set the content type to **application/json**.

![image](https://github.com/user-attachments/assets/c4a0af2a-6cdd-4e92-b500-8a18904317da)


<br/>

![image](https://github.com/user-attachments/assets/af803d98-9560-4874-ada2-32c1bf30e016)




- ✅ If you see a **green check mark**, the webhook is active.
- Any new commits pushed to the repo will automatically trigger the **Jenkins pipeline**.



## Nginx

- [frontend/nginx.conf](frontend/nginx.conf) – Custom Nginx config for frontend deployment
- Serves the Angular app on port 80 and uses Nginx reverse proxy to route /api/ requests to the backend, avoiding CORS issues by keeping all traffic under the same domain.

### Serves at Port 80

- Nginx listens on port 80, making the Angular frontend accessible via http://<server-ip> without needing to specify a port.

<pre lang="markdown">
server {
    listen 80;
    ...
}
</pre>


### API Reverse Proxy

- Forwards all /api/ requests from frontend to the backend container running on `http://backend:8081`.

<pre lang="markdown">
location /api/ {
    proxy_pass http://backend:8081/api/;
    ...
}
</pre>
  
## CI/CD pipeline

![dd_assignment_1](https://github.com/user-attachments/assets/0a8afeb6-6ccc-4970-a8a3-0453eb5119e4)


- As soon as a commit is pushed to the main branch, the configured GitHub webhook triggers the Jenkins pipeline automatically.
- Or you can manually trigger the build from the Jenkins dashboard without any code changes.

### docker images
- Jenkins pipeline automatically builds Docker images during each run.

![image](https://github.com/user-attachments/assets/18793060-8cb9-4dad-87a8-901896c43d58)




### docker hub
- Built and pushed frontend and backend Docker images to Docker Hub for remote storage and deployment.
  
![image](https://github.com/user-attachments/assets/4067862d-7935-4509-ae3f-1891ed79b2c9)


### docker-compose
- Docker Compose pulls images from remote repositories and runs three containers: frontend, backend, and MongoDB, as part of a single service.

![image](https://github.com/user-attachments/assets/911d04d6-3bef-449d-bcb4-25dfcc9cd069)

![image](https://github.com/user-attachments/assets/fc5fe36d-e4a7-4d4e-a01b-2ec29c07e6f7)

### docker volume
- Docker volume is used to persist MongoDB data, ensuring it remains intact across container rebuilds and restarts.

![image](https://github.com/user-attachments/assets/cb0cadd2-f72d-4214-af2e-01a7228f700e)



## Working Application UI

![image](https://github.com/user-attachments/assets/0530899f-581d-4602-8883-ee7af6ec7af4)

![image](https://github.com/user-attachments/assets/c6366cf8-7163-44e4-8066-26c23f52c92d)

![image](https://github.com/user-attachments/assets/86972ba0-830e-4090-aa14-a4202e899fc2)

![image](https://github.com/user-attachments/assets/42e17367-d0aa-4efb-bafb-7be1e02e297c)


<!-- webhook testing 1-->