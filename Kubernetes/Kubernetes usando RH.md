# Curso de Kubernetes implantado en Openshift

### Step 1: Log in to the Developer Sandbox

If you’re unsure how to do this, you can find instructions here: Access your Developer Sandbox for Red Hat OpenShift from the command line | Red Hat Developer

### Step 2: Prepare the local Kubernetes configuration to allow access from the command line

You don't log in to a Kubernetes cluster. You actually set your local environment to access the API server when issuing kubectl commands.

If you have the OpenShift command-line interface (oc) installed (not necessary for this tutorial), you can cheat and use the oc login command. If you can install the oc CLI, it makes life a lot easier.

After establishing those three parts, you use the context you desire. The following commands will take care of this, but first we need to extract some pieces of information from our sandbox. We'll need to get the following items:

- Username, which is represented by {username} in the following commands
- Authorization token {token}
- Name of the cluster {cluster_name}
- Context assigned to us {context}
- URL of the cluster API server {api_server_url}

You'll need to be logged in to your sandbox dashboard to get this information.

**Username**

Our value for {username}. This is displayed in the upper right corner of the dashboard.

**Authorization token**

Our value for {token}. If you click on username, select Copy login command, and log in as DevSandbox, you can see your token.

**URL of the cluster API server**

Our value for {api_server-url}. Using the same instructions as the previous section.

**Name of the cluster**

Our value for {cluster_name}. The cluster name is a modification of the host URL with all periods converted to dashes. Also, the console-openshift-console-apps portion of the host URL is replaced with api. For example, if you navigate to the Topology page of your dashboard, your URL looks there.

**Context assigned to us**

Our value for {context}. The context is constructed by combining your username with the name of the cluster in the following format: {username}-dev/{cluster_name}/{username}.

For example, using what we have up to this point, the value for {context} would be:

```
rhn-engineering-dschenck-dev/api-sandbox-x8i5-p1-openshiftapps-com:6443/rhn-engineering-dschenck
```

Añadimos los siguientes aspectos para la configuración:

```
kubectl config set-credentials {username}/{cluster_name} --token={token}
```

```
kubectl config set-cluster {cluster_name} --server={api_server_url}
```

```
kubectl config set-context {context} --user={username}/{cluster_name} --namespace={username}-dev --cluster={cluster_name}
```

```
kubectl config use-context {context}
```

### Step 3: Clone the source code repositories

git clone https://github.com/redhat-developer-demos/quotesweb.git

git clone https://github.com/redhat-developer-demos/quotemysql.git

git clone https://github.com/redhat-developer-demos/qotd-python.git

### Step 4: Create a back-end program called quotes, then set environment variables to be used by the code

In the directory where you cloned the qotd-python repo, move into the k8s sub-directory and run the following three commands:

```
kubectl create -f quotes-deployment.yaml
```

```
kubectl create -f service.yaml
```

```
kubectl create -f route.yaml
```

Hecho esto, y creados el despliegue, el servicio y la ruta, introducimos el siguiente comando:

```
kubectl get routes
```

```
 curl http://quotes-default.apps.cluster-w5n6d.w5n6d.sandbox199.opentlc.com/quotes
```

**Objects and labels**

When you create the objects for the quotes application, Kubernetes will pull the image from the image registry named in the YAML file and create a pod. It will also assign the labels which you specified in the deployment. The pod name is automatically generated from the deployment name, with random characters appended to it.

Viewing the contents of the file quotes-deployment.yaml, we can see that the containers will be named quotes (plus the random characters, e.g., quotes-5468c95fc6-5sp9j), and the labels will be app: quotes, sandbox: learn-kubernetes, and learn-kubernetes: quotes.

```
kind: Deployment

apiVersion: apps/v1

metadata:

  name: quotes

  labels:

    app: quotes

    sandbox: learn-kubernetes

    sandbox-learn-kubernetes: quotes

spec:

  replicas: 1

  selector:

    matchLabels:

      app: quotes

  template:

    metadata:

      labels:

        app: quotes

    spec:

      containers:

        - name: quotes

          image: quay.io/donschenck/quotes:v1

          imagePullPolicy: Always

          ports:

            - containerPort: 10000

              protocol: TCP
```

Near the end of the YAML file, we can see that the deployment will use the following image:

```
quay.io/rhdevelopers/quotes:v1
```

This is a pre-built Linux image that has data (six "Quote Of The Day"-type entries) hard-coded into the source code. Later, we'll upgrade to version 2, which reads quotes from a MariaDB database (which will also be running in our Kubernetes cluster).

### Step 5: Create a React front-end program called quotesweb

Time to get our front-end quotesweb application up and running in our Kubernetes cluster.

In your quotesweb/k8s directory on your local machine, run the following three commands to create the Deployment, the Service, and the Route:

```
kubectl create -f quotesweb-deployment.yaml
```

```
kubectl create -f quotesweb-service.yaml
```

```
kubectl create -f quotesweb-route.yaml
```

Vamos a la ruta:

```
kubectl get routes
```

```
curl http://quotesweb-default.apps.cluster-w5n6d.w5n6d.sandbox199.opentlc.com/quotesweb
```

Copiamos la url en el navegador para verificar.

### Step 6: Scale the back-end app to two pods and observe the result in quotesweb

At this point, we have two apps (or Kubernetes services) running in our cluster. As you watch the quotesweb application in your browser, you will notice that the hostname is always the same. That's because we have one pod running our quotes service. We can prove this by running the following command (this is optional):

```
kubectl get pods
```

While Kubernetes can be configured to auto-scale (by spinning up additional pods), we can mimic this behavior from the command line and observe the results in our browser. As we increase the number of pods, you'll notice that there are multiple hosts serving quotes.

Run the following command to increase the number of pods to three:

```
kubectl scale deployments/quotes --replicas=3
```

### Step 7: Create a Persistent Volume Claim (PVC) to support MariaDB running in Kubernetes

While many Kubernetes database solutions offer an ephemeral option, that won't suffice for us. We need to make sure the database files remain intact even when the pods running MariaDB are deleted. This requires a PVC.

In your quotemysql directory, you'll find the file mysqlvolume.yaml, and it's 5 GB in size, using the host file system. This is where we'll direct the MariaDB app to place the data files it needs.

```
apiVersion: v1

kind: PersistentVolumeClaim

metadata:

  name: mysqlvolume

spec:

  resources:

    requests:

      storage: 5Gi

  volumeMode: Filesystem

  accessModes:

    - ReadWriteOnce
```

Run the following command to create the PVC:

```
kubectl create -f mysqlvolume.yaml
```

### Step 8: Create a Secret to be use with the database

Navigate to the quotemysql directory on your local PC. You'll find the file mysql-secret.yaml:

```
apiVersion: v1

kind: Secret

metadata:

  name: mysqlpassword

type: Opaque

data:

  password: YWRtaW4=
```

Run the following command to create the Secret object:

```
kubectl create -f mysql-secret.yaml
```

### Step 9: Create a MariaDB database, quotesdb, running in Kubernetes

In your quotemysql directory, you'll find the file mysql-deployment.yaml. Notice the password name (mysqlpassword, the Secret we created in Step 8), the persistentVolumeClaim (mysqlvolume, which we created in Step 7), and the volumeMounts information. These entries should look familiar; we just created them:

```
apiVersion: apps/v1

kind: Deployment

metadata:

  name: mysql

  labels:

    sandbox: learn-kubernetes

    learn-kubernetes: quotemysql

spec:

  selector:

    matchLabels:

      app: mysql

      tier: database

  template:

    metadata:

      labels:

        app: mysql

        tier: database

    spec:

      containers:

      - name: mariadb

        env:

        - name: MYSQL_ROOT_PASSWORD

          valueFrom:

            secretKeyRef:

              name: mysqlpassword

              key: password

        image: mariadb

        resources:

          limits:

            memory: "512Mi"

            cpu: "500m"

        ports:

        - containerPort: 3306

        volumeMounts:

        - name: mysqlvolume

          mountPath: /var/lib/mysql

      volumes:

       - name: mysqlvolume

         persistentVolumeClaim:

           claimName: mysqlvolume
```

We now have all the pieces to spin up a MariaDB database in our Kubernetes cluster: A Persistent Volume Claim, a Secret, and a Deployment.

Run the following command to create the MariaDB database instance:

```
kubectl create -f mysql-deployment.yaml
```

### Step 10: Update the back-end program to version 2 and observe the results in quotesweb

Antes de cambiar a la versión 2, necesitamos hacer algunas tareas de limpieza. Mientras que la versión 1 de nuestro servicio de cotizaciones tiene valores codificados de forma rígida en el código, la versión 2 lee desde el servicio de base de datos mysql. El nombre de este servicio no está codificado en el código fuente. En su lugar, lee el nombre del servicio de la variable de entorno . Aquí está el fragmento de código donde eso sucede:DB_SERVICE_NAME

```
    try:

        conn = mariadb.connect(

            user="root",

            password="admin",

            host=os.environ['DB_SERVICE_NAME'],

            database="quotesdb",

            port=3306)
```

The following command will create that environment variable in our deployment. Then, when we update to version 2, it will be available to the Python code:

```
kubectl set env deployment/quotes DB_SERVICE_NAME=mysql
```

At this point, we have a front-end application (quotesweb) talking to the back-end app (quotes). We also have a database, running in service mysql. What we need to do is update our back-end app to use our database. Kubernetes will do this on the fly, doing what's called a "rolling update." We already have a version 2 image in an image registry, so all we need to do is change the image in our deployment of quotes to point to version 2. Kubernetes will pull the image, spin up a pod running version 2, and then switch the routing to version 2.

Run the following command to switch to version 2:

```
kubectl set image deploy quotes quotes=quay.io/donschenck/quotes:v2
```

After a few seconds—seconds, not minutes—you may need to refresh your browser and re-enter the endpoint URL. At that point, you will notice that there are several more quotes being randomly accessed. Hint: What would happen if you switched back to v1?

### Step 12: Destroy the MariaDB pod to observe Kubernetes' "Self Healing"

Because we use a PVC for our database, instead of an ephemeral database, our data remains intact when a pod falls over. You can prove this by deleting the pod running your MariaDB database. Kubernetes will replace the pod immediately and MariaDB will restart. With no operator intervention. Go ahead, give it a try.

You can remove parts of all of this activity by using one of the following commands:

To remove only the quotes function:

```
kubectl delete pod $PODNAME
```

To remove only the quotesweb application:

```
kubectl delete all -l learn-kubernetes=quotesweb
```

To remove only the MariaDB objects:

```
kubectl delete all -l learn-kubernetes=quotemysql
```

To remove all of the objects associated with this activity:

```
kubectl delete all -l sandbox=learn-kubernetes
```

