
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

## Setup Instructions
0. **Setup the cluster**
   - If you use the cluster from exoscale, you can use this [Infrastructure-as-Code Lab](https://fhb-codelabs.netlify.app/codelabs/iac-opentofu-intro/) to setup the cluster and to get the `kubeconfig`.
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
   - You can use another cluster, if you provide the corresponding `kubeconfig`.

1. **Prerequisites**: 
   - Make sure to have the the current kubeconfig, which stores your cluster information, available as a file named `kubeconfig`.
   - Run following command to have the longhorn storageClass available in your cluster
      ```bash
      kubectl --kubeconfig kubeconfig apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.5.3/deploy/longhorn.yaml
      ```
   - Convert the kubeconfig to base64 and store it in the `staging` or `production` stage in the secret variable `KUBECONFIG_BASE64`.
     Make sure to not copy the empty line from the resulting base64 file.
     - Powershell
         ```powershell
         [Convert]::ToBase64String([IO.File]::ReadAllBytes("kubeconfig")) | Set-Content 'kubeconfig_base64'
         ```
     - Bash
         ```bash
         base64 -w 0 kubeconfig > kubeconfig_base64
         ```
     Please note, this needs to be refreshed after 2 hours, so this is just for testing purposes now.

2. **Deploy MySQL and WordPress**:
   - Execute `kubectl --kubeconfig kubeconfig apply -k ./` to deploy both MySQL and WordPress services in Kubernetes.

3. **Access WordPress**:
   - The github action shows the URL in the summary
   - Manually retrieve the external service IP using `kubectl --kubeconfig kubeconfig get services wordpress`.

## Teardown Instructions
1. **Stop Services**:
   - Use `kubectl --kubeconfig kubeconfig delete -k ./` to stop and remove the WordPress and MySQL deployments from Kubernetes.

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
