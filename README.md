# **Cloud Native Resource Monitoring Python App on K8s!**

 **Cloud Native applications is nothing but building the modern applications that can run in the cloud. These applications are deployed as containers consisting multiple microservices that can be scaled deployed and managed independently, making it more flexible. To achieve this cloud native applications are deployed with an orchestrator such as Kubernetes which can make deployment and management of conatiners very easily.**

## **Prerequisites** !

(Things to have before starting the projects)

- [x]  AWS Account.
- [x]  Programmatic access and AWS configured with CLI.
- [x]  Python3 Installed.
- [x]  Docker and Kubectl installed.
- [x]  Code editor (Vscode)

# ✨Let’s Start the Project ✨

## **Part 1: Deploying the Flask application locally**

### **Step 1: Clone the code**

Clone the code from the repository:

```
git clone <repository_url>
```
### **Step 2: Install Docker,Python,AWS CLI,Kubectl**


Install Kubectl:

curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.11/2023-03-17/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo cp ./kubectl /usr/local/bin
export PATH=/usr/local/bin:$PATH


Install AWScli:

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

![image](https://github.com/user-attachments/assets/a4a5f81a-69d5-4b12-9da6-0d394f6731a7)


### **Step 2: Install dependencies**

The application uses the **`psutil`** and **`Flask`, Plotly, boto3** libraries. Install them using pip:

```
pip3 install -r requirements.txt
```

### **Step 3: Run the application**

To run the application, navigate to the root directory of the project and execute the following command:

```
python3 app.py
```

This will start the Flask server on **`localhost:5000`**. Navigate to [http://localhost:5000/](http://localhost:5000/) on your browser to access the application.

![image](https://github.com/user-attachments/assets/9e152330-7776-4d13-83d1-ba6333f719df)


## **Part 2: Dockerizing the Flask application**

### **Step 1: Create a Dockerfile**

Create a **`Dockerfile`** in the root directory of the project with the following contents:

```
# Use the official Python image as the base image
FROM python:3.9-slim-buster

# Set the working directory in the container
WORKDIR /app

# Copy the requirements file to the working directory
COPY requirements.txt .

RUN pip3 install --no-cache-dir -r requirements.txt

# Copy the application code to the working directory
COPY . .

# Set the environment variables for the Flask app
ENV FLASK_RUN_HOST=0.0.0.0

# Expose the port on which the Flask app will run
EXPOSE 5000

# Start the Flask app when the container is run
CMD ["flask", "run"]
```

### **Step 2: Build the Docker image**

To build the Docker image, execute the following command:

```
docker build -t <image_name> .
```
![image](https://github.com/user-attachments/assets/3fd52b7a-05b6-477c-bd26-6d6e2600e1b4)



### **Step 3: Run the Docker container**

To run the Docker container, execute the following command:

```
docker run -p 5000:5000 <image_name>
```
![image](https://github.com/user-attachments/assets/36cf3c53-54d2-4a60-ab94-a4c45b821279)


This will start the Flask server in a Docker container on **`localhost:5000`**. Navigate to [http://localhost:5000/](http://localhost:5000/) on your browser to access the application.

## **Part 3: Pushing the Docker image to ECR**

### **Step 1: Create an ECR repository**

Create an ECR repository using Python:

```
import boto3

# Create an ECR client
ecr_client = boto3.client('ecr')

# Create a new ECR repository
repository_name = 'my-ecr-repo'
response = ecr_client.create_repository(repositoryName=repository_name)

# Print the repository URI
repository_uri = response['repository']['repositoryUri']
print(repository_uri)
```
![image](https://github.com/user-attachments/assets/e0795ea1-6b28-437c-9159-8275b74c7ce1)

![image](https://github.com/user-attachments/assets/d311be9d-97ab-415d-9c6b-d25ba627d4d8)

### **Step 2: Push the Docker image to ECR**

Push the Docker image to ECR using the push commands on the console:

![image](https://github.com/user-attachments/assets/d6f3c34d-7ad0-4d56-a3c3-7792ecbc95de)


```
docker push <ecr_repo_uri>:<tag>
```

![image](https://github.com/user-attachments/assets/cd6e0093-4e12-410a-95f8-971017de5fe4)

## **Part 4: Creating Roles for Cluster and nodes**

We will create two roles one for AWS EKS or Elastic Kubernetes Services and one for the node group in that cluster.

create a role named as **EKS-Cluster-Role** and in this role attach AmazonEKSClusterPolicy
![image](https://github.com/user-attachments/assets/049e84cf-8c44-4b31-87d3-aee1448c1584)

Create a role for the cluster role group, name it as nodesroles and attach these policies in this role-

AmazonEC2ContainerRegistryReadOnly

AmazonEKS_CNI_Policy

AmazonEKSServicePolicy

AmazonEKSWorkerNodePolicy

AmazonEKSVPCResourceController

![image](https://github.com/user-attachments/assets/6d5c1752-95e6-4ade-989d-c4a127d00131)


## **Part 5: Creating an EKS cluster and deploying the app using Python**

### **Step 1: Create an EKS cluster**

Create an EKS cluster and add node group


![image](https://github.com/user-attachments/assets/095fd4ba-4f38-4327-aa3f-a2eb307b8dad)

![image](https://github.com/user-attachments/assets/cce39276-5a1b-41b9-b62a-34a3307cb1f6)


### **Step 2: Create a node group**

Create a node group in the EKS cluster.

![image](https://github.com/user-attachments/assets/01b2c08c-cf9c-4999-88fd-0e87bfd5d951)





### **Step 3: Create deployment and service**

```jsx
from kubernetes import client, config

# Load Kubernetes configuration
config.load_kube_config()

# Create a Kubernetes API client
api_client = client.ApiClient()

# Define the deployment
deployment = client.V1Deployment(
    metadata=client.V1ObjectMeta(name="my-flask-app"),
    spec=client.V1DeploymentSpec(
        replicas=1,
        selector=client.V1LabelSelector(
            match_labels={"app": "my-flask-app"}
        ),
        template=client.V1PodTemplateSpec(
            metadata=client.V1ObjectMeta(
                labels={"app": "my-flask-app"}
            ),
            spec=client.V1PodSpec(
                containers=[
                    client.V1Container(
                        name="my-flask-container",
                        image="568373317874.dkr.ecr.us-east-1.amazonaws.com/my-cloud-native-repo:latest",
                        ports=[client.V1ContainerPort(container_port=5000)]
                    )
                ]
            )
        )
    )
)

# Create the deployment
api_instance = client.AppsV1Api(api_client)
api_instance.create_namespaced_deployment(
    namespace="default",
    body=deployment
)

# Define the service
service = client.V1Service(
    metadata=client.V1ObjectMeta(name="my-flask-service"),
    spec=client.V1ServiceSpec(
        selector={"app": "my-flask-app"},
        ports=[client.V1ServicePort(port=5000)]
    )
)

# Create the service
api_instance = client.CoreV1Api(api_client)
api_instance.create_namespaced_service(
    namespace="default",
    body=service
)
```

make sure to edit the name of the image on line 25 with your image Uri.

- Once you run this file by running “python3 eks.py” deployment and service will be created.
- Check by running following commands:

```jsx
kubectl get deployment -n default (check deployments)
kubectl get service -n default (check service)
kubectl get pods -n default (to check the pods)
```

Once your pod is up and running, run the port-forward to expose the service

```bash
kubectl port-forward service/<service_name> 5000:5000
```
![image](https://github.com/user-attachments/assets/735e2f29-deee-41ad-a738-284448b93ad8)

