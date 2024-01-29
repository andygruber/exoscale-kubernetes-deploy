
# Kubernetes WordPress Deployment

## Overview
This GitHub repository contains the necessary files for deploying a WordPress site with a MySQL database, managed through Kubernetes. The setup focuses on using a Persistent Volume (PV) for data persistence, making it ideal for test environments like Docker Desktop or Minikube.

It is based of an [example](https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/) from the Kubernetes Documentation.

Other sources are:
- https://chat.openai.com/
- https://hub.docker.com/_/wordpress
- https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
- https://stackoverflow.com/questions/66482385/kubernetes-mysqld-cant-create-write-to-file-var-lib-mysql-is-writable-errc

## Files in the Repository
- `mysql-pv.yaml`: Defines the Persistent Volume for MySQL data storage.
- `mysql-deployment.yaml`: Kubernetes deployment configuration for MySQL.
- `wordpress-deployment.yaml`: Kubernetes deployment configuration for WordPress.
- `kustomization.yaml`: Kustomize configuration to manage Kubernetes objects.
- `.github/workflows`: Github Actions workflows

## Setup Instructions

### Create environment
By default, for all workflow runs, an environment with the name `staging` is used.

Create all required environments and adapt the logic in the workflow (`get_environment -> env_check`) if you want to use multiple environments.

### Required Secrets
- `EXOSCALE_API_KEY`: Exoscale API key
- `EXOSCALE_API_SECRET`: Exoscale API secret

### Required environment variables
- `EXOSCALE_SKS_NAME`: Clustername in exoscale
- `EXOSCALE_ZONE`: Zone where the cluster is deployed, e.g. `at-vie-2`
- `EXOSCALE_USER`: Username in exoscale, is required but does not seem to be used, anything like `admin` does work

### Triggering the Workflow
The workflow is triggered on:
- Pull requests to `main` or `develop` branches
- Published releases
- manually

### Setup the cluster
   - Setup the cluster in Exoscale, you can use this [Infrastructure-as-Code Lab](https://fhb-codelabs.netlify.app/codelabs/iac-opentofu-intro/) to setup the cluster and to get the `kubeconfig`.
     - In that Lab some firewall rules are missing, like described [here](https://community.exoscale.com/documentation/sks/quick-start/#creating-a-cluster-from-the-cli).
       Adapt the example so the loadbalancer works, by adding the following lines to the main.tf file:
       ```
       resource "exoscale_security_group_rule" "nodeportsvc" {
         security_group_id = exoscale_security_group.my_security_group.id
         description       = "Nodeport Services"
         type              = "INGRESS"
         cidr              = "0.0.0.0/0"
         protocol          = "TCP"
         start_port        = 30000
         end_port          = 32767
       }
       ```
   - Run following command to have the longhorn storageClass available in your cluster
      ```bash
      kubectl --kubeconfig kubeconfig apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.5.3/deploy/longhorn.yaml
      ```

## Deploy the Kubernetes project

- Deployment can be done with the Github Action `Deploy Kubernetes project`.
- Local deployment can be done by executing: `kubectl --kubeconfig kubeconfig apply -k ./` 

### Access the Loadbalancer
- The github action shows the URL in the summary
- Manually retrieve the external service IP using `kubectl --kubeconfig kubeconfig get services wordpress`.

## Teardown Instructions
1. **Stop Services**:
   - Deletion can be done with the Github Action `Destroy Kubernetes project`.
   - Locally use `kubectl --kubeconfig kubeconfig delete -k ./` to stop and remove the Kubernetes project.

2. **Delete storageClass**:
   - Run following command to remove the longhorn storageClass from your cluster
      ```bash
      kubectl --kubeconfig kubeconfig delete -f https://raw.githubusercontent.com/longhorn/longhorn/v1.5.3/deploy/longhorn.yaml
      ```

3. Do not forget to destroy the k8s cluster to avoid unnecessary costs.

## Important Notes and Considerations
Add some disclaimers here later

## Contribution and Code Review Process
- Fork the repository to contribute.
- Configure GitHub to require pull request reviews before merging to the main branch.
- Ensure that all code contributions are reviewed and tested by at least one other developer before merging.
