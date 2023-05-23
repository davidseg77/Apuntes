# Curso de Gitops y ArgoCD

En primer lugar, instalamos el operador Red Hat Openshift Gitops.

Pasamos a la consola y efectuamos los siguientes pasos:

Part of the setup of this lab connects you to the Argo CD instance via CLI and UI. Let's find the URL for the ArgoCD API Server:

```
oc get routes -n openshift-gitops | grep openshift-gitops-server | awk '{print $2}'
```

Now, let's store this as an enviroment variable so we can use it later to login to ArgoCD through our terminal.

```
export ARGOCD_SERVER_URL=$(oc get routes -n openshift-gitops | grep openshift-gitops-server | awk '{print $2}')
```

Next, find the default password for the admin account as follows:

```
oc extract secret/openshift-gitops-cluster -n openshift-gitops --to=-
```

To login from the terminal, execute the following, using the $ARGOCD_SERVER_URL we just saved:

```
argocd login $ARGOCD_SERVER_URL
```

You'll have to accept the self-signed certificate.

You can login with the following:

- **Username:** admin
- **Password:** default password from previous command

### The Argo CD Web Console

You can also find this URL by clicking the shortcut from the Application Launcher tab from the OpenShift Web Console:

Nos vamos al cuadrado de la parte superior derecha de la consola.

### ArgoCD Configuration

Let's come back to the terminal. Verify that you're connected to the Argo CD API server by running the following:

```
argocd cluster list
```

You should see output similar to this:

```
SERVER                          NAME        VERSION  STATUS   MESSAGE
https://kubernetes.default.svc  in-cluster           Unknown  Cluster has no application and not being monitored.
```

This output lists the clusters that Argo CD manages. In this case in-cluster in the NAME field signifies that Argo CD is managing the cluster it's installed on.

To enable bash completion, run the following command:

```
source <(argocd completion bash)
```

The Argo CD CLI stores it's configuration under ~/.argocd/config

NOTE The argocd cluster add command used the ~/.kube/config file to establish connection to the cluster.

```
ls ~/.argocd/config
```

The argocd CLI tool is useful for debugging and viewing status of your apps deployed.


