# T-DOP-603 Project NCY_2

## Overview

This project is a Kubernetes-based deployment for a distributed application consisting of multiple services. It includes configurations for monitoring, database management, and ingress routing. The services are deployed using YAML configuration files, ensuring scalability and reliability.

## Project Structure

- **cadvisor.daemonset.yaml**: Deploys cAdvisor for container monitoring.
- **postgres.*.yaml**: Configures PostgreSQL database, volumes, deployment and services.
- **postgres.secret.example.yaml**: Non-sensitive template used to create the local `postgres.secret.yaml` file.
- **redis.*.yaml**: Configures Redis for caching and message queuing.
- **poll.*.yaml**: Deploys the Poll service for user interaction.
- **result.*.yaml**: Deploys the Result service for displaying poll results.
- **worker.deployment.yaml**: Deploys the Worker service for background processing.
- **traefik.*.yaml**: Configures Traefik as the ingress controller.

## Deployment Instructions

### Step 1: Deploy Monitoring

```bash
kubectl apply -f cadvisor.daemonset.yaml
```

### Step 2: Create the local PostgreSQL secret

Copy the tracked example, then replace both placeholder values before applying it:

```bash
cp postgres.secret.example.yaml postgres.secret.yaml
```

`postgres.secret.yaml` is intentionally ignored by Git and must never be committed. The example file contains placeholders only.

Then deploy PostgreSQL:

```bash
kubectl apply -f postgres.secret.yaml -f postgres.configmap.yaml -f postgres.volume.yaml -f postgres.deployment.yaml -f postgres.service.yaml
```

### Step 3: Deploy Redis

```bash
kubectl apply -f redis.configmap.yaml -f redis.deployment.yaml -f redis.service.yaml
```

### Step 4: Deploy Application Services

```bash
kubectl apply -f poll.deployment.yaml -f worker.deployment.yaml -f result.deployment.yaml -f poll.service.yaml -f result.service.yaml -f poll.ingress.yaml -f result.ingress.yaml
```

### Step 5: Deploy Traefik

```bash
kubectl apply -f traefik.rbac.yaml -f traefik.deployment.yaml -f traefik.service.yaml
```

### Step 6: Initialize Database

Create the `votes` table in PostgreSQL:

```bash
echo "CREATE TABLE votes (id text PRIMARY KEY, vote text NOT NULL);" | kubectl exec -i <name_of_the_postgreSQL_pod> -c <name_of_the_container> -- psql -U <postgres_username>
```

### Step 7: Configure Hosts

Update `/etc/hosts` with the service IPs:

```bash
echo "$(kubectl get nodes -o jsonpath='{ $.items[*].status.addresses[?(@.type==\"ExternalIP\")].address }') poll.dop.io result.dop.io" | sudo tee -a /etc/hosts
```

## Notes

- Ensure Kubernetes is properly configured and running before starting the deployment.
- Use `kubectl get pods` and `kubectl logs` to monitor the services.
- Never commit live credentials or generated secret manifests.
