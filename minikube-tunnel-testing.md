# Using Minikube Tunnel with Multi-Site Setup

This document provides detailed instructions on how to use Minikube tunnel to test multiple website deployments with Kustomize.

## What is Minikube Tunnel?

Minikube tunnel creates a network route between your host machine and the Minikube VM, making services of type LoadBalancer and Ingress resources accessible through their defined hostnames. This simulates how your application would work in a production Kubernetes environment.

## Prerequisites

- Minikube installed (v1.20.0 or newer recommended)
- kubectl configured to use your Minikube cluster
- Administrator privileges (for modifying hosts file and running the tunnel)

## Detailed Step-by-Step Guide

### 1. Start Minikube with Sufficient Resources

```bash
minikube start --memory=4096 --cpus=2
```

### 2. Enable the Ingress Addon

The Ingress controller is required to handle hostname-based routing:

```bash
minikube addons enable ingress
```

Verify the Ingress controller is running:

```bash
kubectl get pods -n ingress-nginx
```

You should see pods with names like `ingress-nginx-controller-*` in a Running state.

### 3. Deploy Your Websites

Apply the Kustomize configurations for all websites:

```bash
kubectl apply -k k8s/overlays/website1
kubectl apply -k k8s/overlays/website2
kubectl apply -k k8s/overlays/website3
```

Verify all resources were created:

```bash
kubectl get deployments
kubectl get services
kubectl get configmaps
kubectl get ingress
```

### 4. Start Minikube Tunnel

Open a new terminal window and run:

```bash
minikube tunnel
```

Important notes about tunnel:

- This command must keep running in a separate terminal while you're testing
- It requires administrator privileges (will prompt for password)
- It creates a network route on your host system to the Minikube VM
- The terminal will show log messages about tunnel activity

### 5. Configure Your Hosts File

Since the ingress resources are configured with specific hostnames, you need to map these to your local IP:

1. Open your hosts file with administrator privileges:
   - **Windows**: `C:\Windows\System32\drivers\etc\hosts`
   - **macOS/Linux**: `/etc/hosts`

2. Add these entries:
   ```
   127.0.0.1 website1.local
   127.0.0.1 website2.local
   127.0.0.1 website3.local
   ```

3. Save the file

### 6. Access Your Websites

Now open your browser and navigate to:

- http://website1.local
- http://website2.local
- http://website3.local

Each site should load with its unique content and styling as defined in the respective ConfigMap.

## Understanding How It Works

1. **Ingress Controller**: Acts as a reverse proxy/load balancer that routes requests to the appropriate service based on hostname
2. **Kustomize Overlays**: Each website has a unique configuration that customizes the base deployment
3. **Minikube Tunnel**: Creates a network route so your browser can reach the ingress controller
4. **Hosts File**: Maps the hostnames to your local IP so requests go to Minikube

## Troubleshooting

### Common Issues and Solutions

#### Websites Not Accessible

1. **Verify tunnel is running**:
   Check if the tunnel terminal is still active and showing no errors.

2. **Check ingress resources**:
   ```bash
   kubectl get ingress
   ```
   All should show an ADDRESS value (usually 127.0.0.1).

3. **Verify ingress controller**:
   ```bash
   kubectl get pods -n ingress-nginx
   ```
   The controller pod should be in Running state.

4. **Check service connectivity**:
   ```bash
   kubectl get svc
   ```
   Try port-forwarding to test if services work directly:
   ```bash
   kubectl port-forward svc/website1-website-service 8081:80
   ```

5. **Inspect ingress controller logs**:
   ```bash
   kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx
   ```
   Look for errors related to your ingress resources.

6. **Test DNS resolution**:
   ```bash
   ping website1.local
   ```
   Should resolve to 127.0.0.1.

#### Error: "Could not build host route"

This usually means the tunnel needs administrator privileges. Run the terminal as administrator/with sudo.

## Stopping the Tunnel

When you're done testing:

1. Press `Ctrl+C` in the terminal running the tunnel command
2. The terminal will show cleanup messages as it removes the network routes

## Cleaning Up

To remove all deployed resources:

```bash
kubectl delete -k k8s/overlays/website1
kubectl delete -k k8s/overlays/website2
kubectl delete -k k8s/overlays/website3
```

To stop Minikube completely:

```bash
minikube stop
```
