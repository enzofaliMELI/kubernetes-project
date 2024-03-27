# Kubernetes Project

## Introduction
This repository serves as the project for the Kubernetes course by the Facultad de Ingeniería UdelaR. Within this project, you'll find a web application developed with React that performs real-time calculations of the Fibonacci sequence based on user input.
 
The application is divided into two primary components: a frontend built as a React application and a backend powered by an Express server. The backend includes a specialized worker module that handles the computation of the Fibonacci sequence and stores the results in a Redis database. Moreover, the Express server utilizes a PostgreSQL database to store the historical input values.

The frontend communicates with the backend API, enabling users to access both the current Fibonacci sequence values and the historical input data.

The primary objective of this project is to containerize the application using Docker and deploy it using Kubernetes.

## App Architecture 
This project employs a 3-layer architecture, consisting of the following layers:

1. **Front-end:** This layer is represented by the React app, which serves as the user interface for interacting with the application.

2. **Back-end:** The logic layer is composed of the Express server and the Worker, which handle the application's data processing, and computation of the Fibonacci sequence.

3. **Storage:** The data layer encompasses both the PostgreSQL and Redis databases, where historical input values and computed Fibonacci sequence values are stored, respectively.

<div style="text-align:center">
  <img src="app.jpeg" alt="Alt Text" width="400">
</div>

### Components
1. **Nginx Web Server:** This component routes incoming traffic to various services based on the request type. When accessing the frontend, Nginx directs the request to the React server, and when accessing the backend API, it sends the request to the Express server.

2. **React App (Client):** The frontend of the application, responsible for user interaction.

3. **Express Server:** This server acts as the API layer for the React app, handling requests and responses.

4. **PostgreSQL Database:** This database stores historical input values.

5. **Redis Database:** This database stores the computed Fibonacci sequence values and serves as a queue for the worker to process.

6. **Worker:** This component monitors the next Redis index value and calculates the Fibonacci sequence. Once computed, it stores the results back in Redis. 

## Kubernetes Architecture 

The Kubernetes architecture relies on pods to encapsulate the application containers, promoting scalability and ease of administration. The primary k8 objects utilized are Deployments and Services configured as Cluster IPs.

Deployment objects enable the scaling of client, service, and worker pods as needed. Services, on the other hand, are employed to enable internal network connectivity among the pods. In addition, a Persistent Volume Claim (PVC) is employed for database storage.

Furthermore, the Nginx Ingress Controller serves as the primary gateway for external traffic entering the Kubernetes cluster.  It efficiently handles load balancing and routing, simplifying external access to the cluster's services. This controller also incorporates a default backend for health checks.

<div style="text-align:center">
  <img src="k8s.jpeg" alt="Alt Text" width="650">
</div>

## Project Component Versions

- **Ingress Nginx Controller** - v1.8.2

- **Redis** - v7.2.2

- **Postgres** - v10.1 

The Ingress Nginx Controller is configured based on the documentation available at:
[Ingress Nginx Quick Start Guide](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)
  - This documentation provides instructions on deploying and configuring the Ingress Nginx Controller within a Kubernetes environment.

## Quick Start 

1. Install the Helm Chart:

```bash 
$ helm install webapp-release-v1 webapp/ 
```

2. Install the Ingress Nginx Controller

```bash
$ helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```
or 

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml
```

3. Access the application in your web browser using the following URL:

```bash
http://localhost
```

## Getting Started

### Build Docker Images

Build the Docker images:

```bash
$ docker build -t <name>/worker:<version> ./worker

$ docker build -t <name>/client:<version> ./client

$ docker build -t <name>/server:<version> ./server
```

Replace `<name>` with your Docker image repository name and `<version>` with the desired version or tag for your images.

### Push Docker Images

Push the Docker images to your repository:

```bash
$ docker push <name>/client:<version>

$ docker push <name>/worker:<version>

$ docker push <name>/server:<version>
```

## Kubernetes Deployment

To deploy the application on a Kubernetes cluster using `kubectl`:

Apply the Kubernetes configurations in the "k8" folder to create the necessary resources:

```bash
$ kubectl apply -f k8
```

Check the status of the deployed pods:

```bash
$ kubectl get pods
```

Check the services to ensure they are up and running:

```bash
$ kubectl get services
```

Verify the deployments:

```bash
$ kubectl get deployments
```

Check the Persistent Volume (PV) and Persistent Volume Claim (PVC):
```bash
$ kubectl get pv
$ kubectl get pvc
```

To access the application in your web browser, use the following URL:

```bash
http://localhost
```

### Troubleshooting
 - "context deadline exceeded" error when deploying the Ingress Nginx Controller: 
    - Try redeploying the Ingress Nginx Service using the following command: `kubectl apply -f ingress-service.yaml`
    - Try redeploying the Ingress Nginx Controller using the following command: `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml`
    - Try increasing the timeout value in the Ingress Nginx Controller deployment configuration file.

## Helm Chart

### Prerequisites

- kubectl version: v1.28.2
- helm version: v3.13.0 

### Installing the Chart

To install the Chart with the release name `my-release`:

```bash
$ helm install my-release webapp/ 
```

The comand deploys the web app on the Kubernetes cluster with the default configuration. 

### Installing Ingress Nginx Controller

To deploy the Ingress Nginx Controller using Helm:

```bash
$ helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

or if you prefer to use a YAML manifest: 

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml
```

### List and Status of the Release

To list the releases:  
    
```bash
$ helm list --all-namespaces 
```

To get the status of a release:
```bash
$ helm status <release>
```

### Upgrading the Release

To upgrade the release with a modified configuration, save the new values.yaml file and run the following command:

```bash
$ helm upgrade my-release webapp/ --values webapp/values.yaml
```

### Uninstalling the Chart

To uninstall the Chart with the release name `my-release`:

```bash
$ helm uninstall my-release --keep-history # keep history for rollback purposes
```

## Parameters

The following table lists the configurable parameters of the Web App chart and their default values.

|                 Parameter                 |                                              Description                                               |                           Default                            |
|-------------------------------------------|--------------------------------------------------------------------------------------------------------|--------------------------------------------------------------|
| `Name`                                    | Dynamic Helm Release Naming for Kubernetes Deployments                                                 |                                                              |
| `namespace`                               | Custom Kubernetes Namespace Configuration for Application Deployment                                   | `default`                                                    |
| `client.replicaCount`                     | Client Deployment Pods Number Configuration                                                            | `3`                                                          |
| `client.image.repository`                 | Client Docker Image name                                                                               | `enzofali/multi-client`                                      |
| `client.image.tag`                        | Client Docker Image version                                                                            | `v1`                                                         |
| `server.replicaCount`                     | Server Deployment Pods Number Configuration                                                            | `3`                                                          |
| `server.image.repository`                 | Server Docker Image name                                                                               | `enzofali/multi-server`                                      |
| `server.image.tag`                        | Server Docker Image version                                                                            | `v1`                                                         |
| `worker.replicaCount`                     | Worker Deployment Pods Number Configuration                                                            | `5`                                                          |
| `worker.image.repository`                 | Worker Docker Image name                                                                               | `enzofali/multi-worker`                                      |
| `worker.image.tag`                        | Worker Docker Image version                                                                            | `v1`                                                         |
| `postgres.replicaCount`                   | Postgres Deployment Pods Number Configuration                                                          | `1`                                                          |
| `postgres.image.repository`               | Postgres Docker Image name                                                                             | `postgres`                                                   |
| `postgres.image.tag`                      | Postgres Docker Image version                                                                          | `10.1`                                                       |
| `redis.replicaCount`                      | Redis Deployment Pods Number Configuration                                                             | `1`                                                          |
| `redis.image.repository`                  | Redis Docker Image name                                                                                | `redis`                                                      |
| `redis.image.tag`                         | Redis Docker Image version                                                                             | `7.2.2`                                                      |
| `pvc.accessModes`                         | Persistent Volume Claims Storage Access Mode Configuration                                             | `ReadWriteOnce`                                              |
| `pvc.storageSize`                         | Persistent Volume Claims Storage Size Configuration                                                    | `1Gi`                                                        |
| `service.type`                            | Service Type Configuration for Internal Networking                                                     | `ClusterIP`                                                  |




