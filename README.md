# Deploy Web Application in Kubernetes Cluster from Private Docker Registry (Amazon ECR)

## Project Overview
This project demonstrates how to deploy a web application in a Kubernetes cluster using Docker images hosted in a private Amazon Elastic Container Registry (ECR). It involves creating Secrets for authentication, configuring the Deployment files to reference these Secrets, and deploying the application in Kubernetes.

## Technologies Used
- Kubernetes
- Helm
- AWS ECR
- Docker

## Project Steps
### 1. Create Secrets for Private Docker Registry Credentials
To pull images from a private Docker registry (Amazon ECR), Kubernetes needs access credentials. This can be done using the following methods:

#### Method 1: Using Docker Config File
```bash
kubectl create secret generic my-registry-key \
  --from-file=.dockerconfigjson=/home/irschad/.docker/config.json \
  --type=kubernetes.io/dockerconfigjson
```

#### Method 2: Using ECR Login Credentials
```bash
kubectl create secret docker-registry my-registry-key-two \
  --docker-server=https://922854651834.dkr.ecr.us-east-1.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password --region us-east-1)
```

### 2. Configure the Docker Registry Secret in Application Deployment Files
Two separate Secrets are used for deploying two applications. The Deployment YAML files reference these Secrets to authenticate and pull the application images from the private registry.

#### Deployment Files
- **`myapp-deployment.yaml`**: References `my-registry-key`.
- **`myapp-2-deployment.yaml`**: References `my-registry-key-two`.

Each file specifies the `imagePullSecrets` field to include the appropriate Secret:
```yaml
imagePullSecrets:
  - name: <secret-name>
```

### 3. Deploy Web Application Images

#### Steps:
1. Apply the Secret and Deployment configuration files to the Kubernetes cluster.
   ```bash
   kubectl apply -f docker-secret.yaml
   kubectl apply -f myapp-deployment.yaml
   kubectl apply -f myapp-2-deployment.yaml
   ```

2. Verify the deployment status:
   ```bash
   kubectl get pods
   ```
   Output:
   ```
   NAME                        READY    STATUS   RESTARTS    AGE
   my-app-2-857d6c47d-jpdsz     1/1     Running   0          9m56s
   my-app-8c5f59dc9d-wgsk9      1/1     Running   0          16m
   ```

3. Describe a pod to check if the image was pulled successfully:
   ```bash
   kubectl describe pod <pod-name>
   ```
   ```
   Events:
     Type    Reason     Age   From               Message
     ----    ------     ----  ----               -------
     Normal  Scheduled  17s   default-scheduler  Successfully assigned default/my-app-857d6c47d-jpdsz to minikube
     Normal  Pulling    17s   kubelet            Pulling image "922854651834.dkr.ecr.us-east-1.amazonaws.com/myapp:latest"
     Normal  Pulled     18s   kubelet            Successfully pulled image "922854651834.dkr.ecr.us-east-1.amazonaws.com/myapp:1.0"
     Normal  Created    19s   kubelet            Created container my-app
     Normal  Started    19s   kubelet            Started container my-app
   ```

## Notes
- Ensure that the ECR repository and Kubernetes cluster are in the same AWS region.
- Use `kubectl` to verify that the Secrets are correctly created before deploying.
- Use `kubectl logs <pod-name>` to debug if the application fails to start.

This setup demonstrates best practices for securely deploying applications in Kubernetes using private Docker registries.
