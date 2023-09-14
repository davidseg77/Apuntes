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

### Deploying a Sample Application

In this environment, we have some example manifesets taken from our sample GitOps repo. We'll be uisng this repo to test. These manifests include:

- A Namespace: bgd-ns.yaml
- A Deployment: bgd-deployment.yaml
- A Service: bgd-svc.yaml
- A Route: bgd-route.yaml

Collectively, this is known as an Application within ArgoCD. Therefore, you must define it as such in order to apply these manifest in your cluster.


**Open up the Argo CD Application manifest: bgd-app.yaml**

Let's break this down a bit.

- ArgoCD's concept of a Project is different than OpenShift's. Here you're installing the application in ArgoCD's default project (.spec.project). NOT OpenShift's default project.

- The destination server is the server we installed ArgoCD on (noted as .spec.destination.server).

- The manifest repo where the YAML resides and the path to look for the YAML is under .spec.source.

- The .spec.syncPolicy is set to false. Note that you can have Argo CD automatically sync the repo.

- The last section .spec.sync just says what are you comparing the repo to. (Basically "Compare the running config to the desired config")

The Application CR (CustomResource) can be applied by running the following:

```
oc apply -f https://raw.githubusercontent.com/redhat-developer-demos/openshift-gitops-examples/main/components/applications/bgd-app.yaml
```

This should create the bgd-app in the ArgoCD UI.

Clicking on this "card" takes you to the overview page. You may see it as still progressing or full synced.

At this point the application should be up and running. You can see all the resources created with the command:

```
oc get pods,svc,route -n bgd
```

First wait for the rollout to complete

```
oc rollout status deploy/bgd -n bgd
```

Then visit your application using the route by navigating to the URL under the "HOST/PORT" column

```
oc get route -n bgd
```

Let's introduce a change! Patch the live manifest to change the color of the box from blue to green:

```
oc -n bgd patch deploy/bgd --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/env/0/value", "value":"green"}]'
```

Wait for the rollout to happen:

```
oc rollout status deploy/bgd -n bgd
```

If you refresh your tab where your application is running you should see a green square now.

Looking over at your Argo CD Web UI, you can see that Argo detects your application as "Out of Sync".

You can sync your app via the Argo CD by:

First clicking SYNC
Then clicking SYNCHRONIZE
Conversely, you can run

```
argocd app sync bgd-app
```

After the sync process is done, the Argo CD UI should mark the application as in sync.

If you reload the page on the tab where the application is running. It should have returned to a blue square.

You can setup Argo CD to automatically correct drift by setting the Application manifest to do so. Here is an example snippet:

```
spec:
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Or, as in our case, after the fact by running the following command:

```
oc patch application/bgd-app -n openshift-gitops --type=merge -p='{"spec":{"syncPolicy":{"automated":{"prune":true,"selfHeal":true}}}}'
```







