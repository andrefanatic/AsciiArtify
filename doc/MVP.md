# AsciiArtify MVP Documentation

## Introduction

The MVP will include essential features such as image-to-ascii conversion, user authentication, image uploading, and ascii art display.

## Enable Auto Synchronization

We have not implemented automatic synchronization in POC, but we can enable this option in two ways:

1. With the GUI, click on `Details` of your application and find `SYNC POLICY`
   ![argo-cluster-view](https://github.com/andrefanatic/AsciiArtify/blob/main/doc/img/auto-sync-policy.png)

2. Using the kubectl CLI:
    - first check which synchronization settings are active
    ```
    ➜ kubectl get app asciiartify -n argocd -o=jsonpath='{.spec.syncPolicy}'
    ➜ {"syncOptions":["CreateNamespace=true"]}%
    ```
    - we see that the `CreateNamespace` option is enabled, now we need to execute the following command to enable autosynchronization
    ```
    ➜ kubectl patch app asciiartify -n argocd -p '{"spec": {"syncPolicy": {"automated": {}}}}' --type=merge
    ```
    -command execution description:
    `kubectl patch` This command is used to change part of the configuration of a Kubernetes resource. In our case, we are going to change the configuration of the ArgoCD application.
    `app asciiartify` This is the name of the application we want to change.
    `-n argocd` This option specifies the namespace in which the application is located.
    `-p '{"spec": {"syncPolicy": {"automated": {}}}}'` This is the patch option, which indicates which part of the configuration we want to change. In this case, we are going to change the spec.syncPolicy field, which is responsible for the application's synchronization policy.
    `--type=merge` This parameter specifies how we want to change the configuration. In our case, merge means that we are going to merge our changes with the existing configuration, while keeping other parts of the configuration unchanged.

After this step we won't have to synchronize our K8s deployment with our repository and all changes to the repo will be immediately implemented in the deployment

## Verifying AsciiArtify Application Functionality

1. To verify the functionality of the AsciiArtify application, we need to forward ports:
```
kubectl port-forward -n demo svc/ambassador 8088:80&
```
This service exposes the AsciiArtify application to external traffic within the Kubernetes cluster through the NodePort mechanism. The service is responsible for routing incoming requests to the appropriate backend pods based on the defined rules, allowing external users to access the application's functionalities.

2. And now we can verify that our application works

curl -F 'image=@doc/img/argo.png' localhost:8088/img/

3. Demonstartion
[![asciicast](https://asciinema.org/a/6jMDRyiR6Txm6Byo5PZnh4Dkc.svg)](https://asciinema.org/a/6jMDRyiR6Txm6Byo5PZnh4Dkc)

In conclusion, with a solid foundation in place, empowered by Kubernetes (k8s) and ArgoCD, we are well-positioned to gather feedback from our focus group of users, iterate on the product, and ultimately deliver a solution that exceeds expectations. The streamlined deployment and management provided by Kubernetes and ArgoCD enable us to focus our efforts on product development and innovation, knowing that our infrastructure is robust, scalable, and reliable. We look forward to continuing this journey, collaborating with our users, and shaping the future of AsciiArtify together.