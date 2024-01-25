# Resource-Monitiring(cloud native)-app-k8s
Certainly! Here's the content in Markdown format:

# Part 1: Deploying the Flask application locally

## Step 1: Clone the repository

Clone the project's code by using the following command:

```bash
git clone https://github.com/noobkiing/cloud-python-app-k8s
```

## Step 2: Install dependencies

Ensure you have the required dependencies installed using pip:

```bash
pip3 install -r requirements.txt
```

## Step 3: Run the application locally

Navigate to the project's root directory and run the Flask application:

```bash
python3 app.py
```

Access the application in your browser by going to [http://localhost:5000/](http://localhost:5000/).

# Part 2: Dockerizing the Flask application

## Step 1: Create a Dockerfile

Create a Dockerfile in the project's root directory with the following content:

```Dockerfile
FROM python:3.9-slim-buster

WORKDIR /app

COPY requirements.txt .

RUN pip3 install --no-cache-dir -r requirements.txt

COPY . .

ENV FLASK_RUN_HOST=0.0.0.0

EXPOSE 5000

CMD ["flask", "run"]
```

## Step 2: Build and run the Docker container

Build the Docker image with:

```bash
docker build -t <image_name> .
```

Run the Docker container:

```bash
docker run -p 5000:5000 <image_name>
```

Access the Flask application at [http://localhost:5000/](http://localhost:5000/).

# Part 3: Pushing the Docker image to ECR

## Step 1: Create an ECR repository using Python

Use the following Python script:

```python
import boto3

ecr_client = boto3.client('ecr')

repository_name = 'my-ecr-repo'
response = ecr_client.create_repository(repositoryName=repository_name)

repository_uri = response['repository']['repositoryUri']
print(repository_uri)
```

## Step 2: Push the Docker image to ECR

Execute the following commands:

```bash
docker push <ecr_repo_uri>:<tag>
```


## **Part 4: Creating an EKS cluster and deploying the app using Python**

### **Step 1: Create an EKS cluster**

Create an EKS cluster and add node group

### **Step 2: Create a node group**

Create a node group in the EKS cluster.

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
