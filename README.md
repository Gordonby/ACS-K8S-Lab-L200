# Container Tech Day.


## Command cheat sheet

###Create AKS Cluster
az aks create --resource-group CSADay_K8s_Aks2--name csaDayAks2--node-count 1 --generate-ssh-keys

###Install Kubectl
az aks install-cli

###Get Creds for K8s
az aks get-credentials --resource-group CSADay_K8s_Aks --name csaDayAks2

###Check it's all working
Kubectl get nodes

###Browse k8s dashboard
az aks browse -g CSADay_K8s_Aks -n csaDayAks2 

###Zip up ssh from cloudshell
zip -q -9 -j ~/clouddrive/sshkeys3.zip ~/.ssh/*

###Pull in the ssh files into linux subsystem for windows10
    7  cd /
    8  ls
    9  cd mnt
    10  cd c
    11  ls
    12  cd Users
    13  ls
    14  cd gobyers
    15  ls
    16  cd Downloads
    17  ls
    18  cd sshkeys2
    19  ls
    20  cp -rf * ~/.ssh/

###Install Helm
apt install linuxbrew-wrapper
brew install kubernetes-helm

####Or if that messes up
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh

###Test mongoDb post install
kubectl run mongo-mongodb-client --rm --tty -i --image bitnami/mongodb --command -- mongo --host mongo-mongodb

###Test out by using some simple commands
    show db
    cosmos = { title:"CosmosDb", category:"Database", url:"https://docs.microsoft.com/en-us/azure/cosmos-db/introduction" }
    azuredb = { title:"AzureDb", category:"Database", url:"https://azure.microsoft.com/en-us/services/sql-database/" }
    db.azureservices.save(cosmos)
    db.azureservices.save(azuredb)
    db.azureservices.find()



###Run the ReplicationSet Yaml

###Run the Services Yaml

###Open the swagger page on the new External IP - Submit an order
http://52.178.74.151:8080/swagger/


###Log back into mongo and check for the data

    > db
    test
    > use k8orders
    switched to db k8orders
    > show collections
    orders
    > db.orders.find()
    { "_id" : ObjectId("5a1d8b8faad9887d8fcc124a"), "id" : "5a1d8b8f354a8b004699ab89", "emailaddress" : "test@test.com", "preferredlanguage" : "en-gb", "product" : "balls", "total" : 1, "source" : "swagger", "status" : "Open" }



## Troubleshooting

####HELM refused to work, because of version Mismatch
    helm incompatible versions client[v2.7.2] server[v2.5.1]
>helm init --upgrade



####ERROR: Helm init --upgrade doesn't work
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