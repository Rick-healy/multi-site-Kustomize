# Multi-Site Kustomize Demo - README

This project demonstrates how to manage multiple website configurations on Kubernetes using Kustomize. It provides a practical example of handling different website deployments while minimizing duplication and maintaining consistency.

## Project Structure

```
k8s/
├── base/                 # Base configuration shared by all websites
│   ├── deployment.yaml   # Base deployment spec
│   ├── service.yaml      # Base service spec
│   ├── configmap.yaml    # Base website content
│   ├── ingress.yaml      # Base ingress configuration
│   └── kustomization.yaml
│
└── overlays/             # Site-specific overrides
    ├── website1/         # Website 1 specific configuration
    │   ├── configmap.yaml
    │   ├── ingress.yaml
    │   └── kustomization.yaml
    │
    ├── website2/         # Website 2 specific configuration
    │   ├── configmap.yaml
    │   ├── ingress.yaml
    │   └── kustomization.yaml
    │
    └── website3/         # Website 3 specific configuration
        ├── configmap.yaml
        ├── ingress.yaml
        └── kustomization.yaml
```

## How It Works

1. **Base Configuration**: Contains the common Kubernetes resources (Deployment, Service, ConfigMap, Ingress) that are shared by all websites.

2. **Overlays**: Each website has its own overlay directory with customizations:
   - Unique name prefix (website1-, website2-)
   - Site-specific labels
   - Custom HTML content in the ConfigMap
   - Custom ingress hostname

3. **Kustomize Features Demonstrated**:
   - Name prefixing
   - Common labels
   - Strategic merges for configuration
   - Resource referencing

## Usage

### Prerequisites
- Minikube or any Kubernetes cluster
- kubectl with Kustomize support (built in to recent kubectl versions)

### Setting up Minikube

1. Start Minikube with sufficient resources:
   ```bash
   minikube start --memory=4096 --cpus=2
   ```

2. Enable the Ingress addon (required for hostname-based routing):
   ```bash
   minikube addons enable ingress
   ```

3. Verify the Ingress controller is running:
   ```bash
   kubectl get pods -n ingress-nginx
   ```

### Deploy All Websites

You can deploy all three websites at once:

```bash
kubectl apply -k k8s/overlays/website1
kubectl apply -k k8s/overlays/website2
kubectl apply -k k8s/overlays/website3
```

Verify deployments are running:
```bash
kubectl get pods
kubectl get svc
kubectl get ingress
```

### Access the Websites

You have two options for accessing the websites:

#### Option 1: Using Port Forwarding (Simple but Limited)

```bash
kubectl port-forward svc/website1-website-service 8081:80
kubectl port-forward svc/website2-website-service 8082:80
kubectl port-forward svc/website3-website-service 8083:80
```

Then access:
- http://localhost:8081
- http://localhost:8082
- http://localhost:8083

The downside of this approach is that you need separate terminal windows for each website, and you can't test the Ingress hostname routing.

#### Option 2: Using Minikube Tunnel (Recommended)

Minikube tunnel creates a network route on your host system that allows ingress resources to be accessible through the configured hostnames. This provides a more realistic testing environment.

1. In a **separate terminal window**, start the Minikube tunnel:
   ```bash
   minikube tunnel
   ```
   
   This command needs administrative privileges and must keep running while you're testing. You might see a password prompt.

2. Add the following entries to your hosts file:

   **On Windows**: Edit `C:\Windows\System32\drivers\etc\hosts`
   **On macOS/Linux**: Edit `/etc/hosts`
   
   Add these lines:
   ```
   127.0.0.1 website1.local
   127.0.0.1 website2.local
   127.0.0.1 website3.local
   ```

3. Now you can access all websites through their configured hostnames:
   - http://website1.local
   - http://website2.local
   - http://website3.local

4. To stop the tunnel when you're done testing, press `Ctrl+C` in the terminal window running the tunnel command.

For more detailed instructions on using Minikube tunnel, see the `minikube-tunnel-testing.md` file.

## Extending the Pattern

This simple example can be extended in several ways:

1. **Adding More Websites**: Simply create new overlay directories with their specific configurations.

2. **Environment Support**: Add another layer of overlays for environments (dev/staging/prod).

3. **Shared Components**: Include references to shared services in the base configuration.

4. **Configuration Inheritance**: Create intermediate bases for different website types that extend the main base.

## Benefits of This Approach

- **DRY Principle**: Common configuration is defined once in the base directory.
- **Consistency**: All websites follow the same structure and deployment pattern.
- **Scalability**: Easy to add more websites by creating new overlays.
- **Transparency**: Plain YAML configurations without complex templating logic.
- **Native Integration**: Uses kubectl built-in functionality without additional tools.
