# FlowForge: Real-Time DevOps CI/CD Pipeline with Jenkins, Docker, K8s & ArgoCD


##### An end-to-end DevOps CI/CD project using Jenkins, Docker, Kubernetes, Gitlab and ArgoCD. The aim is to provide a real-time DevOps workflow for building, testing, and deploying using a  Python application.

![Diagram](/Project%20_%20Git%20_%20Jenkins%20_%20docker%20_%20K8s%20_%20Argo.jpeg)

##### Tools Used: *Jenkins*, *Docker*, *Kubernetes*, *ArgoCD*, *Gitlab*


#### Step 1: 

##### Server Installation and Setup

- The project starts with installing Jenkins on an AWS instance, using a provided installation script. The script updates the system, installs Java Runtime Environment (JRE), adds Jenkins repositories, and starts the Jenkins service. The Jenkins server is accessible on port 8080, and the necessary ingress rules are set to allow access.


```bash

#!/bin/bash

sudo apt-get update -y

sudo apt-get install default-jre -y

java -version

sudo apt-get update -y

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update -y

sudo add-apt-repository universe -y

sudo apt-get update -y 

sudo apt-get install jenkins -y

sudo systemctl status jenkins

sudo systemctl start jenkins

cat /var/lib/jenkins/secrets/initialAdminPassword
```

Make sure to set the ingress rule in the security group to allow access to Jenkins on port 8080.

#### Step 2: 

##### Advanced Declarative Jenkinsfile Creation (Groovy Scripting)

The project utilizes Jenkinsfile, written in Groovy scripting, for defining the CI/CD pipeline. The initial Jenkinsfile contains a "Clean workspace" stage, which clears the workspace before starting the build.


- Install the Jenkins Runner plugin in Visual Studio Code (VSCode) and create a settings.json file in the .vscode folder with the following content:

```json

{
  "jenkins-runner.jobs": {
    "test-1": {
      "isDefault": true,
      "runWith": "host-no-password",
      "name": "pipeline-test"
    }
  },
  "jenkins-runner.hostConfigs": {
    "host-with-password": {
      "url": "http://localhost:8090",
      "user": "admin",
      "password": "admin",
      "useCrumbIssuer": true
    }
  }
}
```

#### Step 3: 
##### Jenkins Runner Configuration

- Run Jenkins Runner using the VSCode command palette and execute the Jenkins job using the specified configurations.

#### Step 4: 
##### Trigger Jenkins Job using VSCode itself

- Configure the Jenkins Runner plugin in VSCode to trigger the Jenkins job directly from the editor using the specified configurations.

#### Step 5: 
##### Python Application Overview

- The project includes a simple Python application that demonstrates fetching device details such as hostname, IP address, and MAC address using Flask.

- Create a Python Flask application with the following code:

```python

import socket
from uuid import getnode as get_mac
from flask import Flask, jsonify

# Get device details
def get_device_details():
    hostname = socket.gethostname()
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    s.connect(("8.8.8.8", 80))
    ip = s.getsockname()[0]
    s.close()
    MAC_address = get_mac()
    MAC_address = (':'.join(("%012X" % MAC_address)[i:i+2] for i in range(0, 12, 2))).replace(":", "-")
    return hostname, ip, MAC_address

app = Flask(__name__)

# Returns device hostname, IP, and MAC address
@app.route("/details")
def details():
    hostname, ip, mac = get_device_details()
    out = "Hello!!!....I'm " + hostname + "....My MAC ID is " + mac + "....and My IP address is " + ip
    return out

@app.route("/health")
def health():
    return jsonify(
        status="up"
    )

@app.route("/")
def home():
    return "Welcome to dogad's World"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000), debug=True)

```

#### Step 6: 
##### Docker Image Creation

- The Python application is containerized using Docker. A Dockerfile is provided, which installs Flask and sets up the necessary environment. The Groovy script in the Jenkinsfile is updated to build the Docker image.

- Create a Dockerfile in the project directory with the following content:

```Dockerfile

FROM python:3.9-slim-buster

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]

```

Build the Docker image using the following command:

```bash

docker build -t my-python-app .

```

#### Step 7: 

##### Push the Docker image to Docker Hub 

- Using the following command:

```bash

docker tag my-python-app <username>/my-python-app
docker push <username>/my-python-app

```

#### Step 8: 
##### Kubernetes Cluster Setup

- The project includes Kubernetes manifest files for deploying the Python application as a Kubernetes Deployment and exposing it through a Service. The provided deployment.yml file specifies the desired number of replicas, image, resources, and ports.

- Set up a Kubernetes cluster using Minikube with the following commands:

```bash

minikube start

```

#### Step 9: 
##### Kubernetes Deployment Configuration

- Create a deployment.yaml file with the following content:

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-python-app-deployment
  labels:
    app: my-python-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-python-app
  template:
    metadata:
      labels:
        app: my-python-app
    spec:
      containers:
        - name: my-python-app
          image: <username>/my-python-app
          ports:
            - containerPort: 5000

```

Apply the deployment configuration using the following command:

```bash

kubectl apply -f deployment.yaml

```

#### Step 10: 
##### Expose the Deployment using a Service

- Create a service.yaml file with the following content:

```yaml

apiVersion: v1
kind: Service
metadata:
  name: my-python-app-service
spec:
  selector:
    app: my-python-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: LoadBalancer

```

Apply the service configuration using the following command:

```bash

kubectl apply -f service.yaml

```

#### Step 11: 
##### Install and Configure ArgoCD

- ArgoCD, a continuous deployment tool, is used to manage the Kubernetes deployments. The Kubernetes node is connected to ArgoCD for deploying and managing the application.

- Install ArgoCD in the Kubernetes cluster using the following command:

```bash

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

```

Expose the ArgoCD service using the following command:

```bash

kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

```

Get the ArgoCD password using the following command:

```bash

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

```

#### Step 12: 
##### Deploy the Application using ArgoCD

- Access the ArgoCD dashboard using the external IP:

```bash

kubectl get svc -n argocd argocd-server

```

Login using the admin username and the password obtained in the previous step.

Create an application in ArgoCD with the following specifications:

```yaml

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-python-app
spec:
  destination:
    namespace: default
    server: 'https://kubernetes.default.svc'
  project: default
  source:
    path: my-python-app
      repoURL: <your-git-repo-url>
      targetRevision: HEAD
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

```

Replace <your-git-repo-url> with the URL of your Git repository where the Kubernetes deployment configuration files are stored.

Apply the ArgoCD application configuration using the following command:

```bash

kubectl apply -f argocd-application.yaml

```

#### Step 13: 
##### Verify the Deployment

- Access the application using the external IP of the service exposed by ArgoCD. You should be able to see the Flask application running.

#### Step 14: 
##### Set up Continuous Integration (CI) with Jenkins

Configure Jenkins to monitor the Git repository for changes and trigger the CI pipeline.

-  In Jenkins, create a new pipeline job.
-  Set up the Git repository URL and credentials.
-  Under the Pipeline section, select "Pipeline script from SCM" and provide the Git repository details.
-  Save the configuration.

#### Step 15: 
Set up Continuous Deployment (CD) with Jenkins and ArgoCD
Configure Jenkins to trigger the deployment to Kubernetes using ArgoCD.

- Install the ArgoCD plugin in Jenkins.
- In the Jenkins pipeline job configuration, add the following stage after the CI stage:

```groovy

stage('Deploy to ArgoCD') {
  steps {
    argocd(
      serverUrl: '<argocd-server-url>',
      token: '<argocd-api-token>',
      project: 'default',
      application: 'my-python-app',
      syncOptions: 'Prune=true,SelfHeal=true',
      revision: 'HEAD'
    )
  }
}

```
Replace (argocd-server-url) and (argocd-api-token) with the URL and API token of your ArgoCD server.

Save the configuration.

#### Step 16: 
##### Trigger the CI/CD Pipeline

- Make changes to the application code and push the changes to the Git repository. Jenkins will automatically trigger the CI/CD pipeline.

- The pipeline will build a new Docker image, push it to Docker Hub, update the Kubernetes deployment, and deploy the updated application using ArgoCD.

---

# Congratulations!!!
You have successfully set up a real-time end-to-end DevOps CI/CD pipeline using Jenkins, Docker, Kubernetes, and ArgoCD. You can now make changes to your code, trigger Jenkins pipelines, build Docker images, deploy them to the Kubernetes cluster, and manage the deployments using ArgoCD.

---

## 🔗 Links
[![portfolio](https://img.shields.io/badge/my_portfolio-000?style=for-the-badge&logo=ko-fi&logoColor=white)](https://dolapoadeola.framer.website/)
[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/adeoladolapo/)

---
