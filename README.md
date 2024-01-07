
# Kubernetes WordPress Deployment

## Overview
This GitHub repository contains the necessary files for deploying a WordPress site with a MySQL database, managed through Kubernetes. The setup focuses on using a Persistent Volume (PV) for data persistence, making it ideal for test environments like Docker Desktop or Minikube.

It is based of an [example](https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/) from the Kubernetes Documentation.

## Files in the Repository
- `mysql-pv.yaml`: Defines the Persistent Volume for MySQL data storage.
- `mysql-deployment.yaml`: Kubernetes deployment configuration for MySQL.
- `wordpress-deployment.yaml`: Kubernetes deployment configuration for WordPress.
- `kustomization.yaml`: Kustomize configuration to manage Kubernetes objects.

## Setup Instructions
1. **Create Persistent Volume**: 
   - Run `kubectl apply -f mysql-pv.yaml` to create the Persistent Volume for MySQL.

2. **Deploy MySQL and WordPress**:
   - Execute `kubectl apply -k ./` to deploy both MySQL and WordPress services in Kubernetes.

3. **Access WordPress**:
   - For Docker Desktop users, access WordPress at `http://localhost:80`.
   - Retrieve the external service IP using `kubectl get services wordpress`.

## Teardown Instructions
1. **Stop Services**:
   - Use `kubectl delete -k ./` to stop and remove the WordPress and MySQL deployments from Kubernetes.

2. **Manage Persistent Volume**:
   - Check PV status with `kubectl describe pv`.
   - To make the PV available again, use `kubectl patch pv mysql-pv-volume -p '{"spec":{"claimRef": null}}'`.
   - Alternatively, delete and recreate the PV if data on the host is still available:
     - `kubectl delete -f mysql-pv.yaml`
     - `kubectl apply -f mysql-pv.yaml`

## Important Notes and Considerations
- **Test Environment Suitability**: The current configuration with a static PV using a host-path is suited for test environments like Docker Desktop or Minikube.
- **Adapting for Production**: For production environments, consider dynamic volume provisioning and storage classes for better scalability and management.
- Regularly back up the data stored in the Persistent Volume to prevent data loss.

## Contribution and Code Review Process
- Fork the repository to contribute.
- Configure GitHub to require pull request reviews before merging to the main branch.
- Ensure that all code contributions are reviewed and tested by at least one other developer before merging.
