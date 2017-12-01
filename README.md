# Azure Container Service - Kubernetes Lab.  Level 200-300

## WARNING
This is currently a work in progress....  Do not follow... yet :)

## Origin Story
Forked from https://github.com/shanepeckham/UKTechDay---OSS-App-Modernisation which was a Level 400 version.
The Level 200-300 lab works up the basics a little slower.

## Tools required
Exercises 1-6
* Azure Cloud Shell (Bash version) (It's built into the Azure Portal, nothing to install!)

Exercises 7-15
* Bash on a linux OS (or the Linux subsystem for Windows 10)

## Caveats
No Windows, no GUI's.  By that i mean you can of course use the Windows OS, but please use either the
1. Linux subsystem for Windows 10
1. A Linux VM inside Hyper-V.

If you really want to use the Powershell on Windows 7, go for it... But you'll be going off-lab and will likely have issues to resolve that aren't covered in this guide.

## Exercise 0 - The Illustrated Children's Guide to Kubernetes
If you haven't seen this, watching it will take 8 minutes and it does the best job of talking about the Kubernetes constructs that i've seen.
Go watch it.  Seriously.
https://www.youtube.com/watch?v=4ht22ReBjno 

## Exercise 1 - Creating a Kubernetes Cluster in Azure.
At the time of writing, ACS (Azure Container Service) is the recommended option, not AKS which is still in preview.
There are 2 main ways of creating an ACS Cluster.
1. In the Azure Portal GUI
1. Using the Azure CLI

My preference is the Cloud Shell because it generates the SSH keys for you as well as the service principal.  Feel free to use the Portal GUI, but you're going off-lab (but just a little bit so don't worry too much.

For everyone else, lets open the Cloud Shell.  

> https://portal.azure.com

Login and then click the Cloud Shell icon

![image](./Media/cloud-shell.png) 

You'll then see the Cloud Shell open at the bottom of the screen.

Copy and paste this script into the Cloud Shell.  It'll do 3 things;
1. Create a resource group called K8S in eastus.
1. Create a ACS cluster with a few specific options set.  You don't have to provide half of these, but i quite like the new B- so we're setting that up.  The default would have been a D2_V2 VM for both agent and master VM's that get created.  It also would have been 3 agents, but 1 is enough for the time being.

    ```
    az group create --name K8s --location eastus
    az acs create --orchestrator-type kubernetes --resource-group K8s --name K8sCluster --generate-ssh-keys --agent-count=1 --agent-vm-size=Standard_D2_v2 --agent-osdisk-size=32 --agent-storage-profile=ManagedDisks --master-vm-size=Standard_B1s
    az acs kubernetes get-credentials --resource-group=K8s --name=K8sCluster 
    zip -q -9 -j ~/clouddrive/sshkeys-$(date +%F).zip ~/.ssh/*
    ```

This takes about 10 minutes to provision.  So guess what, that video from step 0 that you didn't watch - go watch it now!  If you actually watched it in Step 0, treat yourself - go watch it again... And remember, always read ahead :)
https://www.youtube.com/watch?v=4ht22ReBjno 

### Post creation
After the cluster has been created, in the Azure Portal find the resource group and open it up.  You'll see a lot of Azure services have been created.  It's important to realise that we have
1. A new vNet
1. VM's, not VM Scalesets
1. A bunch of other infrastructure services

![image](./Media/cluster-created.png) 

## Exercise 2 - kubectl 
Kubectl is the main tool you're going to use to manage your kubernetes cluster.  It comes pre-installed in the Azure Cloud Shell, which is awesome. 

So lets have a look at a basic command that will tell us about the VM's.

```kubectl get nodes```

![image](./Media/get-nodes.png) 

Lets run a couple of other common commands.

```kubectl cluster-info```

![image](./Media/cluster-info.png) 

```kubectl version```

![image](./Media/version.png) 

## Exercise 3 - Creating a simple deployment


## Exercise 4 - Creating a service to expose the pod



    ```
    kubectl get pods
    ```
    ```
    kubectl get svc
    ```


## Exercise 5 - Autohealing

Run this command and see what pods you have running

    ```
    kubectl get pods
    ```

Now delete the pod
    ```
    kubectl delete pod NameOfYourPod
    ```
Run this command again and see what's reported
    ```
    kubectl get pods -w
    ```
I'm expecting that your pod has gone, but a very similarly named one has been put in it's place.
TA-DA.  Autohealing!

## Exercise 6 - Sock shop
Ok, so Exercises 1-5 have been pretty simple.  We're familiar with kubectl, we've created a couple of containers and feeling pretty confident eh?
Now yets run up the Sock Shop, a multi-container deployment which has a web front-end, nosql database and a slew of services.
The purpose of this is
1. Introduction to namespaces
1. Introduction to YAML
1. Introduction to services
1. Introduction to deployments

* You will find the sock shop yaml files inside the SockShop folder. Have at look the various components of the solution and how they are formulated in yaml
* Deploy the objects using:

    ```
    kubectl create -f ./SockShop/
    ```
    ```
    kubectl create -f ./SockShop/deployments
    ```
    ```
    kubectl create -f ./SockShop/services
    ```
    ```
    kubectl create -f ./SockShop/volume
    ```


* You can run the following command against your K8 cluster and see your sock shop pods:

```
kubectl --namespace sock-shop get pods
```

![image](./Media/sock-shop-pods.png) 


Now lets connect to the Sock-Shop front end website.


For now, we're going to abandon Sock Shop, but we'll be back to revisit it.

## Exercise 7 - Contexts, moving and sharing.

By now, we're all bored of Cloud Shell, sure it's nice and easy.. But every time you go for a coffee, it times out - and we need to consider accessing this from somewhere else.

### Contexts
Let's see what contexts are configured.  I've got a tonne, but hopefully you'll just have 1 for the cluster we created in Exercise 1.

    kubectl config view

Now, in the Cloud Shell lets look at where this is kept
    ```cd ~/.kube/```
    ```ls```

You should have *config-K8sCluster-1* listed.
Lets copy that out somewhere safe where we can grab it.
    ```zip -q -9 -j ~/clouddrive/kubeconfig-$(date +%F).zip config-K8sCluster-1```

In the Cloud Shell, lets see where these files are stored.  Click on the *settings* cog and then *Manage File Share* - a new browser window will open in Azure Files where you can easily navigate to the directory with the kubeconfig zip file in it to download to your machine.

### SSH Keys
We should backup the SSH keys that were generated when we created the cluster.  You'll need these kept somewhere safe when your running K8S in a real environment.

    zip -q -9 -j ~/clouddrive/sshkeys-$(date +%F).zip ~/.ssh/*

In the same way as you downloaded the config zip, do the same for the ssh key zip.

### Installing Kubectl
We need to install Kubectl into your linux environment.

If you're using *Bash on Ubuntu on Windows* here's the command

    ```
    .
    ```

Otherwise, consult this page on how to install it : https://kubernetes.io/docs/tasks/tools/install-kubectl/


### Copying in the config
Copy it to the directory /.kube on your machine, it should exist if you instal kubectl. 
Now to connect to the cluster run the folling command

```kubectl --kubeconfig ./<your cluster config file> get nodes```

And now make yourself an alias for brevity:

```alias k8='kubectl --kubeconfig ./<your cluster config file>'```

Now you can run the following command:

```k8 get nodes```



## Exercise 8 - Accessing the dashboard
Kubernetes has a web dashboard, who knew!
Lets have a look

```az acs kubernetes browse -g K8S -n K8SCluster```

## Exercise 9 - Creating a deployment in the Dashboard GUI
From within the dashboard, go add
1. new pod
1. name =nginx
1. external ip
1. port 80
1. boom.

Let's back out of the dashboard, now and for the rest of the lab.

Run this command to see what's been created.

    ```
    kubectl get pods
    ```
    ```
    kubectl get svc
    ```

## Exercsie 10 - Helm
Helm is a package manager for Kubernetes.  It will simply allow the installation of complex software using *Helm Charts* (are you noticing a terminology theme yet :)).

Please read the introduction section through to the installation section here : https://github.com/kubernetes/helm

Once installed, initialise it.

    ```
    helm init
    ```

Now lets have a quick look at all the things we could install with Helm

    ```
    helm search
    ```
    

## Exercise 11 - MongoDb and Persistent Volume Claims
Running a database in a container... A good idea?  Lets see.

Here we want you to 
1. Deploy a mongoDB single pod using a standard Helm chart

To install MongoDB via Helm use

```helm install --name mongo stable/mongodb```

If you get an error about tiller version, update the HELM version on the server, upgrade using:

```helm init --upgrade```

To see the environment variables
```kubectl exec -it <thenameofyourpod> -- /bin/bash```
```
printenv
```

## Exercise 12 - Running an API to add data to MongoDb
1. Deploy the container *captureordertd:v2*, populating the environment variables and expose the captureordertd:v2 container as a public endpoint by using a Service. (*Yaml writing time everyone!  Notepad (or VS Code at the ready)*)
1. You must deploy v2 of the captureordertd

* Your captureordertd service has an external IP
* Inspect the environment variables to make sure your team name is correct 
* You can connect to the captureordertd API via the following uri http://<your svc IP>:8080/swagger
* You can run the Swagger test harness and get a MongoDB orderid as a response
* You can connect to your Helm installed MongoDB instance using *mongo-mongodb-client* and check for a record
* Check the pod logs to inspect the operations
* Your team name is passed into the captureordertd container - *you will receive no scores if you do not do this*

### Hints

See https://hub.docker.com/r/shanepeckham/captureordertd/ for information on the container and what the environment variables look like for the v2 tag. You need to map this container over port 8080 as this is where it is listening.


## Exercise 13 - Yaml and Environment Variables

## Exercise 14 - Load testing

## Exercise 15 - Horizontal Scale Out
In Kubernetes an HPA is a horizontal scaling unit with which we will use to auto-scale a Deployment based on threshold criteria. Your challenge is to run a load test against the captureordertd endpoint and see your captureordertd pods autoscale.

* You must deploy captureordertd as a Deployment object in Kubernetes in order to apply an HPA to it
* You can use the following on cluster container to test the load on your captureordertd Deployment
* You must deploy v2 of the captureordertd


```
kubectl run -i --tty load-generator --image=busybox /bin/sh
```

If it is already running, you can attach to it:

```
kubectl attach <pod> -c load-generator -i -t
```

And then run the following infinite load generation script:

```
while true; do wget -q -O- http://<externalIp or internal DNS>:8080/v1/order/ --post-data "EmailAddress=abc@abc.com"; done
```

Use the following to see your top pod activity

```
kubectl top pods 
```

and to see your pod status as they inflate and deflate

```
kubectl get pods -w
```

### Success Criteria

* You are able to see the multiple pods spawning during the load

## Exercise 16 - Perform a rolling update - live replace of MongoDB with CosmosDB 

We now need to replace MongoDB with CosmosDB, without minimal downtime. Fortunately, we have a MongoDB API driver on CosmosDB. You need to deploy V3 of the captureordertd by means of a rolling update and still be able to capture orders, now in CosmosDB

### Hints

* You must deploy v3 of the captureordertd and replace v2 - see Docker Hub for any info
* You must provision a CosmosDB instance and use the MongoDB API driver

### Success Criteria

* You can see records captured to CosmosDB not MongoDB
* You can see your pods being replaced by the new version
* You can rollback the update - see below

Once you have completed the upgrade, do the following:

To see the status of the rolling update

```
kubectl rollout history deployment captureordertd
```

Now rollback the update using this command

```
kubectl rollout undo deployment captureordertd
```







## Command cheat sheet

### Create AKS Cluster
```
az acs create --orchestrator-type kubernetes --resource-group <yourresourcegroupk8> --name <yourk8cluster> --generate-ssh-keys
```

### Install Kubectl
```az acs kubernetes install-cli``

### Get Creds for K8s
```az acs kubernetes get-credentials --resource-group=<yourresourcegroupk8> --name=<yourk8cluster>```

###Check it's all working
```kubectl get nodes```

### Browse k8s dashboard

```az acs kubernetes browse -g <yourresourcegroupk8> -n <yourk8cluster>```

### Install Helm
```
apt install linuxbrew-wrapper
brew install kubernetes-helm
```

#### Or if that messes up
```
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```

### Test mongoDb post install
```
kubectl run mongo-mongodb-client --rm --tty -i --image bitnami/mongodb --command -- mongo --host mongo-mongodb
```

### Test out by using some simple commands
```
    show dbs
    cosmos = { title:"CosmosDb", category:"Database", url:"https://docs.microsoft.com/en-us/azure/cosmos-db/introduction" }
    azuredb = { title:"AzureDb", category:"Database", url:"https://azure.microsoft.com/en-us/services/sql-database/" }
    db.azureservices.save(cosmos)
    db.azureservices.save(azuredb)
    db.azureservices.find()
```


### Open the swagger page on the new External IP - Submit an order
http://<your external ip>:8080/swagger/


### Log back into mongo and check for the data

    > db
    test
    > use k8orders
    switched to db k8orders
    > show collections
    orders
    > db.orders.find()
    { "_id" : ObjectId("5a1d8b8faad9887d8fcc124a"), "id" : "5a1d8b8f354a8b004699ab89", "emailaddress" : "test@test.com", "preferredlanguage" : "en-gb", "product" : "balls", "total" : 1, "source" : "swagger", "status" : "Open" }

### Inspect pod logs
    kubectl logs thenameofyourpod

## Troubleshooting

#### HELM refused to work, because of version Mismatch
    helm incompatible versions client[v2.7.2] server[v2.5.1]
>helm init --upgrade



#### ERROR: Helm init --upgrade doesn't work
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    @         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    Permissions 0777 for '/root/.ssh/id_rsa' are too open.
    It is required that your private key files are NOT accessible by others.
    This private key will be ignored.
    Load key "/root/.ssh/id_rsa": bad permissions
    Permission denied (publickey).

> chmod 600 /root/.ssh/id_rsa

> ssh azureuser@k8scheapcl-k8scheap-2d5bb2mgmt.westeurope.cloudapp.azure.com sudo sed -i s/'2.5.1'/'2.7.2'/g /etc/kubernetes/addons/kube-tiller-deployment.yaml && helm init --upgrade
    
    $HELM_HOME has been configured at /root/.helm.
    Tiller (the Helm server-side component) has been upgraded to the current version.
    Happy Helming!



####ERROR: Can't retrieve the Server version of HELM after upgrade
    root@MININT-90OLH41:~# helm version
    Client: &version.Version{SemVer:"v2.7.2", GitCommit:"8478fb4fc723885b155c924d1c8c410b7a9444e6", GitTreeState:"clean"}
    Error: cannot connect to Tiller

>REBOOT THE VM's!