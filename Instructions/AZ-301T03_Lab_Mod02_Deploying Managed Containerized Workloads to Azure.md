# Creating Managed Server Applications in Azure

# Lab Answer Key: Deploying Managed Containerized Workloads to Azure

## Before we start

1. Ensure that you are logged in to your Windows 10 lab virtual machine using the following credentials:

    - Username: **Admin**

    - Password: **Pa55w.rd**

1. Review Taskbar located at the bottom of your Windows 10 desktop. The Taskbar contains the icons for the common applications you will use in the labs:

    - Microsoft Edge

    - File Explorer

    - [Visual Studio Code](https://code.visualstudio.com/)

    - [Microsoft Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/)

    - Bash on Ubuntu on Windows

    - Windows PowerShell

    > **Note**: You can also find shortcuts to these applications in the **Start Menu**.

## Exercise 1: Create Azure Kubernetes Service (AKS) cluster

#### Task 1: Open the Azure Portal

1. On the Taskbar, click the **Microsoft Edge** icon.

1. In the open browser window, navigate to the **Azure Portal** (<https://portal.azure.com>).

1. If prompted, authenticate with the user account account that has the owner role in the Azure subscription you will be using in this lab.

#### Task 2: Open Cloud Shell

1. At the top of the portal, click the **Cloud Shell** icon to open a new shell instance.

    > **Note**: The **Cloud Shell** icon is a symbol that is constructed of the combination of the *greater than* and *underscore* characters.

1. If this is your first time opening the **Cloud Shell** using your subscription, you will see a wizard to configure **Cloud Shell** for first-time usage. When prompted, in the **Welcome to Azure Cloud Shell** pane, click **Bash (Linux)**.

    > **Note**: If you do not see the configuration options for **Cloud Shell**, this is most likely because you are using an existing subscription with this course's labs. If so, proceed directly to the next task.

1. In the **You have no storage mounted** pane, click **Show advanced settings**, perform the following tasks:

    - Leave the **Subscription** drop-down list entry set to its default value.

    - In the **Cloud Shell region** drop-down list, select the Azure region matching or near the location where you intend to deploy resources in this lab.

    - In the **Resource group** section, ensure that the **Create new** option is selected and then, in the text box, type **AADesignLab0401-RG**.

    - In the **Storage account** section, ensure that the **Create new** option is selected and then, in the text box below, type a unique name consisting of a combination of between 3 and 24 characters and digits.

    - In the **File share** section, ensure that the **Create new** option is selected and then, in the text box below, type **cloudshell**.

    - Click the **Create storage** button.

1. Wait for the **Cloud Shell** to finish its first-time setup procedures before you proceed to the next task.

#### Task 3: Create an AKS cluster by using Cloud Shell

1. At the **Cloud Shell** command prompt at the bottom of the portal, type in the following command and press **Enter** to create a variable which value designates the name of the resource group you will use in this task:

    ```sh
    RESOURCE_GROUP='AADesignLab0402-RG'
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the Azure region you will use for the deployment (replace the placeholder `<Azure region>` with the name of the Azure region to which you intend to deploy resources in this lab. `az account list-locations` will list all available locations for your subscription.):

    ```sh
    LOCATION='<Azure region>'
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a new resource group:

    ```sh
    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a new AKS cluster:

    ```sh
    az aks create --resource-group $RESOURCE_GROUP --name aad0402-akscluster --node-count 1 --node-vm-size Standard_D2s_v3 --generate-ssh-keys
    ```

    > **Note**: If you receive an error message regarding availability of the VM size which value is represented by the `--node-vm-size` parameter, review the message and try other suggested VM sizes.

    > **Note**: Alternatively, in **PowerShell** on **Cloud Shell**  you can identify VM sizes available in your subscription in a given region by running the following command and reviewing the values in the **Restriction** column (make sure to replace the `region` placeholder with the name of the target region):

    ```pwsh
    Get-AzComputeResourceSku | where {$_.Locations -icontains "region"} | Where-Object {($_.ResourceType -ilike "virtualMachines")}
    ```

    > **Note**: The **Restriction** column will contain the value **NotAvailableForSubscription** for VM sizes that are not available in your subscription.

1. Wait for the deployment to complete before you proceed to the next task.

    > **Note**: This operation can take up to 10 minutes.


#### Task 4: Connect to the AKS cluster.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to retrieve the credentials to access the AKS cluster:

    ```sh
    az aks get-credentials --resource-group $RESOURCE_GROUP --name aad0402-akscluster
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to verify connectivity to the AKS cluster:

    ```
    kubectl get nodes
    ```

1. At the **Cloud Shell** command prompt, review the output and verify that the node is reporting the **Ready** status. Rerun the command until the correct status is shown.

> **Result**: After you complete this exercise, you should have successfully deployed a new AKS cluster.


## Exercise 2: Managing an AKS cluster and its containerized workloads.

#### Task 1: Deploy a containerized application to an AKS cluster

1. In the Microsoft Edge window, in the Azure portal, at the **Cloud Shell** prompt, type the following command and press **Enter** in order to deploy the **nginx** image from the Docker Hub:

    ```
    kubectl create deployment aad0402-akscluster --image=nginx --replicas=1 --port=80
    ```

    > **Note**: Make sure to use lower case letters when typing the name of the deployment. You will also receive a notification that this command is deprecated and will be removed in a future version, but successfully created the cluster.
    
1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a deployment:

    ```
    kubectl create deployment aad0402-akscluster --image=nginx --replicas=1 --port=80
    ```
1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to verify that a Kubernetes pod has been created:

    ```
    kubectl get pods
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to identify the state of the deployment:

    ```
    kubectl get deployment
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to make the pod available from Internet:

    ```
    kubectl expose deployment aad0402-akscluster --port=80 --type=LoadBalancer
    ```

    > **Note**: Make sure to use lower case letters when typing the name of the deployment.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to identify whether the public IP address has been provisioned:

    ```
    kubectl get service --watch
    ```

1. Wait until the value in the **EXTERNAL-IP** column for the **aad0402-akscluster** entry changes from **\<pending\>** to a public IP address, then press **Ctrl-C** key combination. Note the public IP address in the **EXTERNAL-IP** column for **aad0402-akscluster**.

1. Start Microsoft Edge and browse to the IP address you obtained in the previous step. Verify that Microsoft Edge displays a web page with the **Welcome to nginx!** message.


#### Task 2: Scaling containerized applications and AKS cluster nodes

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to scale the deployment:

    ```
    kubectl scale --replicas=2 deployment/aad0402-akscluster
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to verify the outcome of scaling the deployment:

    ```
    kubectl get pods
    ```

    > **Note**: Review the output of the command and verify that the number of pods increased to 2.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to scale out the number of cluster nodes:

    ```sh
    az aks scale --resource-group $RESOURCE_GROUP --name aad0402-akscluster --node-count 2
    ```

1. Wait for the provisioning of the additional node to complete.

    > **Note**: This operation can take up to 10 minutes. If it fails, rerun the `az aks scale` command.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to verify the outcome of scaling the cluster:

    ```
    kubectl get nodes
    ```

    > **Note**: Review the output of the command and verify that the number of nodes increased to 2.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to scale the deployment:

    ```
    kubectl scale --replicas=10 deployment/aad0402-akscluster
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to verify the outcome of scaling the deployment:

    ```
    kubectl get pods
    ```

    > **Note**: Review the output of the command and verify that the number of pods increased to 10.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to review the pods distribution across cluster nodes:

    ```
    kubectl get pod -o=custom-columns=NODE:.spec.nodeName,POD:.metadata.name
    ```

    > **Note**: Review the output of the command and verify that the pods are distributed across both nodes.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to delete the deployment:

    ```
    kubectl delete deployment aad0402-akscluster
    ```

## Exercise 3: Autoscaling pods in an AKS cluster

#### Task 1: Deploy a Kubernetes pod by using a **.yaml** file.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to download a sample containerized application:

    ```
    git clone https://github.com/Azure-Samples/azure-voting-app-redis.git
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to navigate to the location of the downloaded app:

    ```sh
    cd azure-voting-app-redis
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to list the content of the application **.yaml** file:

    ```sh
    cat azure-vote-all-in-one-redis.yaml
    ```

1. Review the output of the command and verify that the pod defninition includes requests and limits in the followng format:

    ```yaml
    resources:
      requests:
        cpu: 250m
      limits:
        cpu: 500m
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to deploy the application based on the **.yaml** file:

    ```
    kubectl apply -f azure-vote-all-in-one-redis.yaml
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to to verify that a Kubernetes pod has been created:

    ```
    kubectl get pods
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to identify whether the public IP address for the containerized application has been provisioned:

    ```
    kubectl get service azure-vote-front --watch
    ```

1. Wait until the value in the **EXTERNAL-IP** column for the **azure-vote-front** entry changes from **\<pending\>** to a public IP address, then press **Ctrl-C** key combination. Note the public IP address in the **EXTERNAL-IP** column for **azure-vote-front**.

1. Start Microsoft Edge and browse to the IP address you obtained in the previous step. Verify that Microsoft Edge displays a web page with the **Azure Voting App** message.

#### Task 2: Autoscale Kubernetes pods.

1. At the **Cloud Shell** command prompt, type in the following commands and press **Enter** after each to change the current directory and download a sample containerized application:

    ```
    cd ..
    git clone https://github.com/kubernetes-incubator/metrics-server.git
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to install **Metrics Server**:

    ```
    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to configure autoscaling for the **azure-vote-front** deployment:

    ```
    kubectl autoscale deployment azure-vote-front --cpu-percent=50 --min=3 --max=10
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to view the status of autoscaling:

    ```
    kubectl get hpa
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to view the pods:

    ```
    kubectl get pods
    ```

    > **Note**: Verify that the number of replicas increased to 3. If that is not the case, wait one minute and rerun the two previous steps.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to delete the deployment:

    ```
    kubectl delete deployment azure-vote-front
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to delete the deployment:

    ```
    kubectl delete deployment azure-vote-back
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to verify that the commands you ran in the previous steps completed successfully:

    ```
    kubectl get pods
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to delete the AKS cluster:

    ```
    az aks delete --resource-group AADesignLab0402-RG --name aad0402-akscluster --yes --no-wait
    ```

1. Close the **Cloud Shell** pane.

> **Review**: In this exercise, you implemented autoscaling of pods in an AKS cluster


## Exercise 4: Implement DevOps with AKS

#### Task 1: Deploy DevOps with AKS

> **Note**: This solution is based on the DevOps with Containers solution described at https://docs.microsoft.com/en-us/azure/architecture/example-scenario/apps/devops-with-aks.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to generate the SSH key pair that will be used to authenticate when accessing the Linux VMs running the Jenkins instance and Grafana console:

    ```sh
    ssh-keygen -t rsa -b 2048
    ```

    - When prompted to enter the file in which to save the key, press **Enter** to accept the default value **(~/.ssh/id_rsa)**.

    - When prompted to enter passphrase, press **Enter** twice.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the public key of the newly generated key pair:

    ```sh
    PUBLIC_KEY=$(cat ~/.ssh/id_rsa.pub)
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the public key of the newly generated key pair and which takes into account any special character the public key might include:

    ```sh
    PUBLIC_KEY_REGEX="$(echo $PUBLIC_KEY | sed -e 's/\\/\\\\/g; s/\//\\\//g; s/&/\\\&/g')"
    ```

    > **Note**: This is necessary because you will use the **sed** utility to insert this string into the Azure Resource Manager template parameters file. Alternatively, you could simply open the file and enter the public key string directly into the file.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the name of the resource group you will use for the deployment:

    ```sh
    RESOURCE_GROUP='AADesignLab0403-RG'
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the Azure region you will use for the deployment:

    ```sh
    LOCATION=$(az group list --query "[?name == 'AADesignLab0402-RG'].location" --output tsv)
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a new resource group:

    ```sh
    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create an Azure Active Directory service principal for the authentication of services and resources within the sample solution:

    ```sh
    SERVICE_PRINCIPAL=$(az ad sp create-for-rbac --name http://AADesignLab0403-SP --output json)
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to retrieve the **appId** attribute of the newly created service principal:

    ```sh
    APP_ID=$(echo $SERVICE_PRINCIPAL | jq -r .appId)
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to retrieve the **password** attribute of the newly created service principal:

    ```sh
    PASSWORD=$(echo $SERVICE_PRINCIPAL | jq -r .password)
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create the parameters file you will use for deployment of the sample solution and open it in the vi interface:

    ```sh
    vi ~/parameters.json
    ```

1. At the **Cloud Shell** command prompt, in the vi editor interface, add the content of the sample parameters file (**\\allfiles\\AZ-301T03\\Module_02\\Labfiles\\Starter\\parameters.json**):

    ```json
    {
      "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "spClientId": {
          "value": "$APP_ID"
        },
        "spClientSecret": {
          "value": "$PASSWORD"
        },
        "linuxAdminUsername": {
          "value": "Student"
        },
        "linuxAdminPassword": {
          "value": "Pa55w.rd1234"
        },
        "linuxSSHPublicKey": {
          "value": "$PUBLIC_KEY_REGEX"
        }
      }
    }
    ```

1. At the **Cloud Shell** command prompt, in the vi editor interface, to save your changes and close the file, press **Esc**, press **:**, type **wq!** and press **Enter**.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to replace the placeholder for the **appId** attibute with the value of the **$APP_ID** variable in the parameters file:

    ```sh
    sed -i.bak1 's/"$APP_ID"/"'"$APP_ID"'"/' ~/parameters.json
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to replace the placeholder for the **password** attribute with the value of the **$PASSWORD** variable in the parameters file:

    ```sh
    sed -i.bak2 's/"$PASSWORD"/"'"$PASSWORD"'"/' ~/parameters.json
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to replace the placeholder for the **sshPublicKey** parameter with the value of the **$PUBLIC_KEY_REGEX** variable in the parameters file:

    ```sh
    sed -i.bak3 's/"$PUBLIC_KEY_REGEX"/"'"$PUBLIC_KEY_REGEX"'"/' ~/parameters.json
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to verify that the placeholders were successfully replaced in the parameters file:

    ```sh
    cat ~/parameters.json
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to identify AKS versions supported in the Azure region you are using in this lab:

    ```sh
    az aks get-versions --location $LOCATION --output table
    ```
    
1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to deploy the sample solution by using its Azure Resource Manager template residing in a GitHub repository:

    ```sh
    az group deployment create --resource-group $RESOURCE_GROUP --template-uri https://raw.githubusercontent.com/MicrosoftLearning/AZ-301-MicrosoftAzureArchitectDesign/master/allfiles/AZ-301T03/Module_02/LabFiles/Starter/azuredeploy.json --parameters @parameters.json
    ```
    > **Note**: If prompted, provide one of the currently supported AKS versions.
    
    > **Note**: If you receive an error message regarding availability of the VM size which value is represented by the `--node-vm-size` parameter, review the message and try other suggested VM sizes.

    > **Note**: Alternatively, in **PowerShell** on **Cloud Shell**  you can identify VM sizes available in your subscription in a given region by running the following command and reviewing the values in the **Restriction** column (make sure to replace the `region` placeholder with the name of the target region):

    ```pwsh
    Get-AzComputeResourceSku | where {$_.Locations -icontains "region"} | Where-Object {($_.ResourceType -ilike "virtualMachines")}
    ```

    > **Note**: The **Restriction** column will contain the value **NotAvailableForSubscription** for VM sizes that are not available in your subscription.
    
1. Wait for the deployment to complete before you proceed to the next task.

    > **Note**: The deployment can take up to 15 minutes.


#### Task 2: Review the DevOps with AKS architecture

1. In the hub menu in the Azure portal, click **Resource groups**.

1. On the **Resource groups** blade, click the entry representing the **AADesignLab0403-RG** resource group.

1. On the **AADesignLab0403-RG** resource group blade, review the list of resources and compare them with the information available at https://docs.microsoft.com/en-us/azure/architecture/example-scenario/apps/devops-with-aks

> **Review**: In this exercise, you deployed DevOps with AKS architecture.


## Exercise 5: Remove lab resources

#### Task 1: Open Cloud Shell

1. At the top of the portal, click the **Cloud Shell** icon to open the Cloud Shell pane.

1. At the **Cloud Shell** command prompt at the bottom of the portal, type in the following command and press **Enter** to list all resource groups you created in this lab:

    ```sh
    az group list --query "[?starts_with(name,'AADesignLab04')]".name --output tsv
    ```

1. Verify that the output contains only the resource groups you created in this lab. These groups will be deleted in the next task.

#### Task 2: Delete resource groups

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to delete the resource groups you created in this lab

    ```sh
    az group list --query "[?starts_with(name,'AADesignLab04')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

1. Close the **Cloud Shell** prompt at the bottom of the portal.


> **Review**: In this exercise, you removed the resources used in this lab.
