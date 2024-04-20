# Proof of Concept for AsciiArtify: Deployment of ArgoCD

## Introduction

This document provides instructions for setting up and accessing ArgoCD, a GitOps continuous delivery tool, as part of the Proof of Concept (PoC) for AsciiArtify.

## Prerequisites

- Access to a Kubernetes cluster (deployed using k3d as recommended in Concept.md)
- kubectl configured to communicate with the Kubernetes cluster

## Steps to Deploy ArgoCD

1. **Deploy ArgoCD to Kubernetes:**
   
   Execute the following command to deploy ArgoCD to your Kubernetes cluster:
   ```
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

2. **Expose ArgoCD Server:**

   By default, the ArgoCD Server service is of type `ClusterIP`, which is only accessible within the Kubernetes cluster. To access it locally, you can use port forwarding. Execute the following command to forward the ArgoCD server port to your local machine:
   ```
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```

3. **Access ArgoCD UI:**

   So, we have configured a proxy server to forward traffic from local port 8080 to port 443 (port 443 is used for HTTPS connections) of the "argocd-server" service in the "argocd" namespace. You should now be able to access the Argo CD using your browser by going to:
   ```
   http://localhost:8080
   ```

4. **Login to ArgoCD:**

   - Username: `admin`
   - Password: Retrieve the default admin password using the following command:
     ```
     kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
     ```

# Deploying Application with ArgoCD GUI

Let's create an application using the graphical interface. Now, applications configured in ArgoCD will automatically install and update in Kubernetes.

5. **Create an application using the graphical interface. Now, configured applications in ArgoCD will automatically install and update in Kubernetes.**

   - Click + NEW APP
   ![argo-dashsboard](https://github.com/andrefanatic/AsciiArtify/blob/main/doc/img/argo-dash-1.png)

   - Enter the application name demo
   - Select the project to which the application belongs by default
   - Leave the synchronization type as Manual
   - In the SYNC OPTIONS section, specify how the application will synchronize with the repository. Here it is important to instruct ArgoCD to create a new namespace since Helm removed this feature by default. Check AUTO-CREATE NAMESPACE.

   ![argo-app-creation](https://github.com/andrefanatic/AsciiArtify/blob/main/doc/img/argo-newapp-1.png)


   - In the SOURCE section, leave the source type as GIT
   - Enter the repository URL containing the manifests for deployment, for example, https://github.com/andrefanatic/go-ascii-app (there will be Helm charts, or a package of manifests representing a group of Kubernetes objects for our application)
   - In the Path field, enter the path to the helm directory "helm"

   - In the DESTINATION section, specify the URL of the local cluster and the demo Namespace, after which ArgoCD will automatically determine the application parameters using the manifests located in the repository. If desired, you can manually change their values in the PARAMETERS section.

   ![argo-app-creation](https://github.com/andrefanatic/AsciiArtify/blob/main/doc/img/argo-newapp-2.png)

   - Create the application by clicking CREATE

6. **Review the details of the deployed application by clicking on it in the list.**

   - The graphical interface provides a hierarchical view of program components, their deployment, and the current state in the cluster.
   ![argo-cluster-view](https://github.com/andrefanatic/AsciiArtify/blob/main/doc/img/argo-cluster-view.png)


7. **Application synchronization**

   - To do this, in the application details window, click SYNC
   - On the right, a window will appear where you need to select the components and synchronization modes, then click SYNCHRONIZE
   - After the process is complete, you can verify the correctness of the application deployment by checking its status in the cluster.
   ![argo-app-synchronization](https://github.com/andrefanatic/AsciiArtify/blob/main/doc/img/argo-sync-2.png)

8. **Observe ArgoCD's reaction to changes in the repository.**

   - Change the type of gateway from LoadBalancer to NodePort in the helm values.file at the repository https://github.com/andrefanatic/go-ascii-app/blob/master/helm/values.yaml (last line of the file)
   ![argo-repo-update](https://github.com/andrefanatic/AsciiArtify/blob/main/doc/img/update-source-repo.png)

   - The synchronization process initiated will fetch the latest version from the git repository and compare it with the current state. Thus, we see that the service type for ambassador has changed from NodePort to LoadBalancer, and accordingly, the Kubernetes manifest has been updated.
   ![argo-app-resync](https://github.com/andrefanatic/AsciiArtify/blob/main/doc/img/argo-out-of-sync.png)

## Conclusion

Following these steps will deploy and provide access to ArgoCD, allowing AsciiArtify's team to begin utilizing GitOps practices for continuous delivery. This Proof of Concept demonstrates the technical feasibility of integrating ArgoCD into AsciiArtify's workflow.
