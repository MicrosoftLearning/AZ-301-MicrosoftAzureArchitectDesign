# Networking Azure Application Components

# Lab Answer Key: Deploying Network Infrastructure for Use in Azure Solutions

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
    
1. Ensure that you have 12 available Standard DSv2 Family vCPUs cores in the region that you will use in the lab. If this is not the case, follow the procedure described in [Standard quota: Increase limits by VM series](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/per-vm-quota-requests) to increase the quotas.

## Exercise 1: Configure the lab environment

#### Task 1: Open the Azure Portal and Cloud Shell.

1. On the Taskbar, click the **Microsoft Edge** icon.

1. In the open browser window, navigate to the **Azure Portal** (<https://portal.azure.com>).

1. When prompted, authenticate with the user account account that has the owner role in the Azure subscription you will be using in this lab.

1. At the top of the portal, click the **Cloud Shell** icon to open a new shell instance.

    > **Note**: The **Cloud Shell** icon is a symbol that is constructed of the combination of the *greater than* and *underscore* characters.

1. If this is your first time opening the **Cloud Shell** using your subscription, you will see a wizard to configure **Cloud Shell** for first-time usage. When prompted, in the **Welcome to Azure Cloud Shell** pane, click **Bash (Linux)**.

    > **Note**: If you do not see the configuration options for **Cloud Shell**, this is most likely because you are using an existing subscription with this course's labs. If so, proceed directly to the next task.

1. In the **You have no storage mounted** pane, click **Show advanced settings**, perform the following tasks:

    - Leave the **Subscription** drop-down list entry set to its default value.

    - In the **Cloud Shell region** drop-down list, select the Azure region matching or near the location where you intend to deploy resources in this lab

    - In the **Resource group** section, select the **Create new** option and then, in the text box below, type **AADesignLab0801-RG**.

    - In the **Storage account** section, ensure that the **Create new** option is selected and then, in the text box below, type a unique name consisting of a combination of between 3 and 24 characters and digits.

    - In the **File share** section, ensure that the **Create new** option is selected and then, in the text box below, type **cloudshell**.

    - Click the **Create storage** button.

1. Wait for the **Cloud Shell** to finish its first-time setup procedures before you proceed to the next task.

#### Task 2: Deploy an Azure VM running Linux Ubuntu 18.04 that will be used to perform Azure Building Blocks-based deployments.

> **Note**: This is necessary to account for breaking changes affecting running from Cloud Shell.

1. At the **Cloud Shell** command prompt, run the following to create a variable which value designates the name of your Azure subscription:

1. At the **Cloud Shell** command prompt, run the following to create a variable which value designates the name of the resource group you will use in this exercise:

    ```sh
    RESOURCE_GROUP='AADesignLab0802-RG'
    ```

1. At the **Cloud Shell** command prompt, run the following to create a variable which value designates the Azure region you will use for the deployment (replace the `<location>` placeholder with the name of the Azure region you want to use in this lab):

    ```sh
    LOCATION='<location>'
    ```

1. At the **Cloud Shell** command prompt, run the following to create a resource group that you will use for the deployment:

    ```sh
    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

1. At the **Cloud Shell** command prompt, run the following to deploy an Azure VM running Linux Ubuntu 18.04 that you will use for deploying resources via Azure Building Blocks:

    ```sh
    UBUNTU_IMAGE='Canonical:UbuntuServer:18.04-LTS:latest'
    VM_NAME='lab08vm1'
    USER_NAME='student'
    az vm create \
    --name $VM_NAME \
    --resource-group $RESOURCE_GROUP \
    --location $LOCATION \
    --image $UBUNTU_IMAGE \
    --admin-username $USER_NAME \
    --generate-ssh-keys \
    --size Standard_DS1_v2
    ```

> **Note**: Wait until the deployment completes.

1. At the **Cloud Shell** command prompt, run the following to retrieve the public IP address of the newly deployed Azure VM **lab08vm1**:

    ```sh
    IP_ADDRESS=$(az vm show -d --resource-group $RESOURCE_GROUP --name $VM_NAME --query publicIps -o tsv)
    ```

1. In the **Cloud Shell** pane, run the following to open an SSH session to the newly deployed Azure VM **lab08vm1**:

    ```sh
    ssh student@$IP_ADDRESS
    ```

1. In the **Cloud Shell** pane, when prompted whether to continue, type **yes** and press Enter.

#### Task 3: Install the Azure Building Blocks npm package in the Azure VM running Linux.

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to update locally installed packages and their dependencies:

    ```sh
    sudo apt-get update
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to ensure that there are no remaining updates to be processed:

    ```sh
    sudo -i apt upgrade
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to install Azure CLI: 

    ```sh
    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to install node.js v10: 

    ```sh
    curl -sL https://deb.nodesource.com/setup_10.x | sudo bash -
    sudo apt-get install -y nodejs
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to verify the versions of node.js and npm:

    ```sh
    node --version
    npm --version
    ```

    > **Note**: Verify that the versions are **v10.21.0** and **6.14.4**, respectively.

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to install Azure Building Blocks:

    ```sh
    sudo su - 
    npm install -g @mspnp/azure-building-blocks
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to exit the root mode and switch to the home directory:

    ```sh
    exit
    cd $HOME
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to set the npm path: 

    ```sh
    npm config set prefix '~/.npm-global'
    ```

#### Task 4: Prepare Building Blocks Hub and Spoke parameter files

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to download the GitHub repository containing the Azure Building Blocks reference architecture files:

    ```
    git clone https://github.com/mspnp/reference-architectures.git
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to view the content of the Azure Building Block parameter file you will use for this deployment:

    ```sh
    cat ./reference-architectures/hybrid-networking/hub-spoke/hub-spoke.json
    ```
    
1. Within the SSH session to the Azure VM **lab08vm1**, run the following to change the current directory to the one hosting the **hub-spoke.json** file you will use for this deployment:

    ```sh
    cd ./reference-architectures/hybrid-networking/hub-spoke
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, type in the following command and press **Enter** to create a new file:

    ```sh
    vi hub-spoke.json.patch
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, in the vi editor interface, type **a** to initiate the **INSERT** mode, press **Enter** to start a new line, and then type the following:

    ```sh
    --- hub-spoke.json.orig	2020-06-11 16:08:04.175635600 +0200
    +++ hub-spoke.json	2020-06-11 16:12:07.494153300 +0200
    @@ -271,27 +271,6 @@
                             {
                                 "resourceGroupName": "spoke1-vnet-rg",
                                 "vmCount": 1,
    -                            "namePrefix": "s1jb",
    -                            "computerNamePrefix": "s1jb",
    -                            "adminUsername": "[replace-with-username]",
    -                            "adminPassword": "[replace-with-password]",
    -                            "osType": "windows",
    -                            "virtualNetwork": {
    -                                "resourceGroupName": "spoke1-vnet-rg",
    -                                "name": "spoke1-vnet"
    -                            },
    -                            "nics": [
    -                                {
    -                                    "isPublic": false,
    -                                    "subnetName": "mgmt",
    -                                    "privateIPAllocationMethod": "Static",
    -                                    "startingIPAddress": "10.1.0.36"
    -                                }
    -                            ]
    -                        },
    -                        {
    -                            "resourceGroupName": "spoke1-vnet-rg",
    -                            "vmCount": 1,
                                 "namePrefix": "s1jbl",
                                 "computerNamePrefix": "s1jbl",
                                 "adminUsername": "[replace-with-username]",
    @@ -330,27 +309,6 @@
                                         "startingIPAddress": "10.2.0.36"
                                     }
                                 ]
    -                        },
    -                        {
    -                            "resourceGroupName": "spoke2-vnet-rg",
    -                            "vmCount": 1,
    -                            "namePrefix": "s2jbl",
    -                            "computerNamePrefix": "s2jbl",
    -                            "adminUsername": "[replace-with-username]",
    -                            "adminPassword": "[replace-with-password]",
    -                            "osType": "linux",
    -                            "virtualNetwork": {
    -                                "resourceGroupName": "spoke2-vnet-rg",
    -                                "name": "spoke2-vnet"
    -                            },
    -                            "nics": [
    -                                {
    -                                    "isPublic": false,
    -                                    "subnetName": "mgmt",
    -                                    "privateIPAllocationMethod": "Static",
    -                                    "startingIPAddress": "10.2.0.37"
    -                                }
    -                            ]
                             }
                         ]
                     },
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, in the vi editor interface, press **Esc**, press **:**, type **wq!** and press **Enter** to save your changes and close the file.

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to patch the **hub-spoke.json** (two virtual machines have to be removed due to a quota limitation in the Free Azure Pass subscription - 10 virtual machines in total):

    ```sh
    patch < hub-spoke.json.patch
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to replace the placeholder **[replace-with-username]** with the value **Student** in the **hub-spoke.json** Building Blocks parameter file:

    ```sh
    sed -i.bak1 's/\[replace-with-username\]/Student/g' ./hub-spoke.json
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to replace the placeholder **[replace-with-password]** with the value **Pa55w.rd1234** in the **hub-spoke.json** Building Blocks parameter file:

    ```sh
    sed -i.bak2 's/\[replace-with-password\]/Pa55w.rd1234/g' ./hub-spoke.json
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to replace the placeholder **[replace-with-shared-key]** parameter with the value **shared12345** in the **hub-spoke.json** Building Blocks parameter file:

    ```sh
    sed -i.bak3 's/\[replace-with-shared-key\]/shared12345/g' ./hub-spoke.json
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to verify that the parameter values were successfully changed in the **hub-spoke.json** Building Blocks parameter file:

    ```sh
    cat ./hub-spoke.json
    ```

#### Task 5: Implement the hub component of the Hub and Spoke design

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to authenticate to your Azure subscription:

    ```sh
    az login
    ```

    > **Note**: Follow the instructions provided in the output of the command to authenticate to your Azure subscription.

1. Once you authenticated, return to the Cloud Shell pane displaying the SSH session to the Azure VM **lab08vm1**, and run the following to create a variable which value designates the name of your Azure subscription:

    ```sh
    SUBSCRIPTION_ID=$(az account list --query "[0].id" --output tsv | tr -d '"')
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to create a variable which value designates the name of the resource group that will contain the hub virtual network:

    ```sh
    RESOURCE_GROUP=onprem-vnet-rg
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to create a variable which value designates the Azure region you will use for the deployment (replace the placeholder `<Azure region>` with the name of the Azure region to which you intend to deploy resources in this lab):

    ```sh
    LOCATION='<Azure region>'
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to deploy the hub component of the Hub-and-Spoke topology by using the Azure Building Blocks:

    ```sh
    azbb -s  $SUBSCRIPTION_ID -g $RESOURCE_GROUP -l $LOCATION -p ./hub-spoke.json --deploy
    ```

1. Wait for the deployment to complete but proceed to the next task.

    > **Note**: The deployment can take about 40 minutes.
    
1. In case the Cloud Shell session timed out, reopen it, reestablish the SSH session to the Azure VM **lab08vm1**, and run the following to authenticate to your Azure subscription:

    ```sh
    az login
    ```

    > **Note**: Follow the instructions provided in the output of the command to authenticate to your Azure subscription.

1. Within the SSH session to the Azure VM **lab08vm1**, and run the following to create a variable which value designates the name of your Azure subscription:

    ```sh
    SUBSCRIPTION_ID=$(az account list --query "[0].id" --output tsv | tr -d '"')
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to create a variable which value designates the name of the resource group that will contain the hub virtual network:

    ```sh
    RESOURCE_GROUP=onprem-vnet-rg
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to create a variable which value designates the Azure region you will use for the deployment (replace the placeholder `<Azure region>` with the name of the Azure region to which you intend to deploy resources in this lab):

    ```sh
    LOCATION='<Azure region>'
    ```

1. Within the SSH session to the Azure VM **lab08vm1**, run the following to change the current directory to the one hosting the **hub-spoke.json** file you will use for this deployment:

    ```sh
    cd ./reference-architectures/hybrid-networking/hub-spoke
    ```
    
1. Within the SSH session to the Azure VM **lab08vm1**, run the following to deploy the hub component of the Hub-and-Spoke topology by using the Azure Building Blocks:

    ```sh
    azbb -s  $SUBSCRIPTION_ID -g $RESOURCE_GROUP -l $LOCATION -p ./hub-firewall.json --deploy
    ```

1. Wait for the deployment to complete but proceed to the next task.

    > **Note**: The deployment can take about 2 minutes.   


## Exercise 2: Review the Hub-spoke topology

#### Task 1: Examine the peering configuration

1. In the hub menu in the Azure portal, click **All services**.

1. In the **All services** menu, in the **Filter** text box, type **Virtual networks** and press **Enter**.

1. In the list of results, click **Virtual networks**.

1. On the **Virtual networks** blade, click **hub-vnet**.

1. On the **hub-vnet** blade, click **Peerings**.

1. On the **hub-vnet - Peerings** blade, review the list of peerings and their status.

1. Navigate back to the **Virtual Networks** blade and click **spoke1-vnet**.

1. On the **spoke1-vnet** blade, click **Peerings**.

1. On the **spoke1-vnet - Peerings** blade, review the existing peering and its status.

1. Navigate back to the **Virtual Networks** blade and click **spoke2-vnet**.

1. On the **spoke2-vnet** blade, click **Peerings**.

1. On the **spoke2-vnet - Peerings** blade, review the existing peering and its status.

#### Task 2: Examine the routing configuration

1. In the **All services** menu, in the **Filter** text box, type **Route tables** and press **Enter**.

1. In the list of results, click **Route tables**.

1. On the **Route tables** blade, click **spoke1-rt**.

1. On the **spoke1-rt** blade, review the list of routes. Note the **NEXT HOP** entry for the route **toSpoke2**.

1. Navigate back to the **Route tables** blade and click **spoke2-rt**.

1. On the **spoke2-rt** blade, review the list of routes. Note the **NEXT HOP** entry for the route **toSpoke1**.

#### Task 3: Verify connectivity between spokes

1. In the hub menu in the Azure portal, click **All services**.

1. In the **All services** menu, in the **Filter** text box, type **Network Watcher** and press **Enter**.

1. In the list of results, click **Network Watcher**.

1. On the **Network Watcher** blade, in the **NETWORK DIAGNOSTIC TOOLS** section, click **Connection troubleshoot**.

1. On the **Network Watcher - Connection troubleshoot** blade, perform the following tasks:

    - Leave the **Subscription** drop-down list entry set to its default value.

    - In the **Resource group** drop-down list, select the **AADesignLab08-spoke1-vnet-rg** entry.

    - In the **Virtual machine** drop-down list, leave the default entry.

    - Ensure that the **Destination** option is set to **Specify manually**.

    - In the **URI, FQDN, or IPv4** text box, type **10.2.0.68** entry.

    - In the **Destination Port** text, type 3389.

    - Click the **Check** button.

1. Wait until results of the connectivity check are returned and verify that the status is **Reachable**.

    > **Note**: If this is the first time you are using Network Watcher, the check can take up to 5 minutes.


## Exercise 3: Remove lab resources

#### Task 1: Open Cloud Shell

1. At the top of the portal, click the **Cloud Shell** icon to open the Cloud Shell pane.

1. In the Cloud Shell pane, run the following command and press **Enter** to list all resource groups you created in this lab:

    ```sh
    az group list --query "[?starts_with(name,'AADesignLab08')]".name --output tsv
    ```

1. Verify that the output contains only the resource groups you created in this lab. These groups will be deleted in the next task.

#### Task 2: Delete resource groups

1. In the Cloud Shell pane, run the following command and press **Enter** to delete the resource groups you created in this lab

    ```sh
    az group list --query "[?starts_with(name,'AADesignLab08')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

1. Close the **Cloud Shell** prompt at the bottom of the portal.

> **Review**: In this exercise, you removed the resources used in this lab.
