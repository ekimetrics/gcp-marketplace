# gcp-marketplace
google marketplace

# Application Deployment Guide on GKE

This guide explains how to deploy your application on Google Kubernetes Engine (GKE) using a web-based deployment form. The form gathers all necessary information to configure and deploy your application successfully.

## Getting Started

Go to: [Google Cloud Marketplace](https://console.cloud.google.com/marketplace/product/eu-prd-ekimetrics-ekimktplace/one.vision?project=eu-prd-ekimetrics-ekimktplace)

And then click on "Configure"

To deploy your application, you can choose to either deploy it on an existing Kubernetes cluster or create a new one through the form. Here are the steps and fields you need to complete:

### Deployment Options

- **Click to Deploy on GKE**: Use this option to deploy directly through the Google Cloud Platform (GCP) interface.
- **Deploy via Command Line**: Use this option if you prefer to deploy using command line tools like `kubectl`.

### Cluster Configuration

- **Existing Kubernetes Cluster**: Select this if you already have a cluster where you want to deploy the application.
- **Create a New Cluster**: Select this to set up a new cluster specifically for this application.

### Basic Information

- **Namespace**: The Kubernetes namespace where the application will be deployed. Typically set to `default` unless you have a specific namespace for organization.
- **App Instance Name**: The name for your application instance. This name will identify your deployment within the cluster.

### Application Configuration

#### Cluster Requirements

- **Cluster Requirements**: Confirm your cluster meets the necessary requirements for the application, such as memory and specific node configurations. This is typically done by entering "YES" if compliant.

#### License and Database Information

- **Licence Key**: Your specific license key for the application.
- **MongoDB URI Format**: The URI format to connect to your MongoDB instance, such as `mongodb+srv://`.

#### MongoDB Database Details

- **MongoDB Database Common User**: Username for accessing the common database.
- **MongoDB Database Common Password**: Password for the common database user.
- **MongoDB Database User Modeling**: Username for accessing the modeling-specific database.
- **MongoDB Database Modeling Password**: Password for the modeling database user.
- **MongoDB Common Database Host**: Hostname or IP address of the common MongoDB server.
- **MongoDB Modeling Database Host**: Hostname or IP address of the modeling MongoDB server.
- **MongoDB Database Port**: Port number used to connect to your MongoDB servers.

#### Database Naming

- **MongoDB Common Database Name**: The database name for the common database.
- **MongoDB MODELING-ENV Database Name**: The database name for the modeling environment.

### Service Integration Tokens and Secrets

- **POSTMARK Token**: Your token for integrating Postmark services for email sending.
- **JWT Secret**: Secret key used for encoding and decoding JWT tokens for authentication purposes.

### Azure and GCP Configurations

- **Azure AD Admin Group ID**: The identifier for your admin group in Azure AD.
- **AZURE TENANT ID**: Your Azure tenant ID.
- **Azure App Client ID**: Client ID for your Azure application.
- **Azure App Client Secret**: Secret key for your Azure application.
- **GCP IAP Client ID**: Client ID for Google Cloud Platform's Identity-Aware Proxy, used for securing applications.
- **GCP IAP Client Secret**: Secret key for the GCP IAP.

### URL Configuration

- **URL Hostname**: The hostname where your application will be accessed.
- **URL Domain Name**: The domain name under which your application will be hosted.

### Deploy Button

- **Deploy**: After filling all the required fields, click this button to start the deployment process.

## Configuring Kubernetes Ingress Post-Deployment

### Introduction

After deploying your application to Google Kubernetes Engine (GKE), setting up Kubernetes Ingress is crucial for routing external traffic to the correct services within your cluster. This section provides detailed instructions on how to configure Ingress resources.

### Prerequisites

Before proceeding, ensure:

- Your Kubernetes cluster is active.
- Your application is deployed within the cluster.
- `kubectl` is installed and configured.
- Nginx Ingress Controller is installed in your cluster.

### Ingress Configuration

#### Common Service Ingress

The following configuration is for routing traffic to the common services of your application. It includes settings for handling standard HTTP requests as well as WebSocket connections.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: common-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/use-regex: 'true'
    nginx.ingress.kubernetes.io/proxy-body-size: 10m
    nginx.ingress.kubernetes.io/client-body-buffer-size: 10m
    nginx.ingress.kubernetes.io/server-snippet: |
      location (?*)socket\.io(?*) {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Host $http_host;
        proxy_set_header Sec-WebSocket-Protocol $http_sec_websocket_protocol;
        proxy_set_header Sec-WebSocket-Extensions $http_sec_websocket_extensions;
        proxy_set_header Sec-WebSocket-Key $http_sec_websocket_key;
        proxy_set_header Sec-WebSocket-Version $http_sec_websocket_version;
      }
spec:
  ingressClassName: nginx
  rules:
    - host: "{{.Values.ingress_url}}.{{.Values.domain_name}}"
      http:
        paths:
          - path: /?(.*)
            pathType: Prefix
            backend:
              service:
                name: common-front
                port:
                  number: 80
          - path: /common/{1}?(.*)
            pathType: Prefix
            backend:
              service:
                name: common-back
                port:
                  number: 3000
```

#### Modeling Service Ingress

This configuration routes traffic to your modeling-specific backend services, configured similarly to the common services but targeted to different paths and services.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: modeling-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/use-regex: 'true'
    nginx.ingress.kubernetes.io/proxy-body-size: 10m
    nginx.ingress.kubernetes.io/client-body-buffer-size: 10m
    nginx.ingress.kubernetes.io/server-snippet: |
      location (?*)socket\.io(?*) {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Host $http_host;
        proxy_set_header Sec-WebSocket-Protocol $http_sec_websocket_protocol;
        proxy_set_header Sec-WebSocket-Extensions $http_sec_websocket_extensions;
        proxy_set_header Sec-WebSocket-Key $http_sec_websocket_key;
        proxy_set_header Sec-WebSocket-Version $http_sec_websocket_version;
      }
spec:
  ingressClassName: nginx
  rules:
    - host: "onevision.ekimetrics.io"
      http:
        paths:
          - path: /{{ .Values.modelingEnvName }}/{1}(.*)
            pathType: Prefix
            backend:
              service:
                name: modeling-env-back
                port:
                  number: 3000
```

#### Applying the Configuration

To apply these Ingress configurations:

Save each configuration block to a YAML file.
Deploy the configuration using kubectl apply -f [filename].yaml for each file.
These steps will establish routing rules for your application, allowing you to manage incoming traffic based on the URL paths specified in the Ingress configuration.

### Conclusion

Ensure that all fields are filled accurately to prevent issues during deployment. 
This setup will guide the application through a correct and efficient deployment process, integrating with necessary external services and configuring according to your specified requirements.
