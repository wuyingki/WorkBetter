# Provision an Oracle Container Engine for Kubernetes

## Preparing Container Engine for Kubernetes

Before you can use Container Engine for Kubernetes to create a Kubernetes cluster:

- You must have access to an Oracle Cloud Infrastructure tenancy

- Your tenancy must have sufficient quota on different types of resource
  - at least one compute instance (node) must be available in the tenancy or three compute instances (one in each availability domain) to create a highly available cluster
  - at least one load balancer to distribute traffic between the nodes running a service in a Kubernetes cluster

- A suitably pre-configured compartment must already exist in each region in which you want to create and deploy clusters. If you are using a GSE environment, then you should use the **Demo** compartment and **api.user** as the local OCI login account. Otherwise, use a local user account that's been created for you by your administrator.
  - The compartment must contain the necessary network resources already configured (VCN, subnets, internet gateway, route table, security lists) before you can create and deploy a cluster. For example, to create a highly available cluster spanning three availability domains, the VCN must include three subnets in different availability domains for node pools, and two further subnets for load balancers.

- Within the root compartment of your tenancy, a policy statement (allow service OKE to manage all-resources in tenancy) must be defined to give Container Engine for Kubernetes access to resources in the tenancy.

- To create and/or manage clusters, your user account must belong to one of the following:
  - The tenancy's Administrators group
  - A group to which a policy grants the appropriate Container Engine for Kubernetes permissions

- To perform operations on a cluster:
  - You must have installed and configured the Kubernetes command line tool `kubectl`
  - You must have downloaded the cluster's `kubeconfig` file

The following steps illustrates how you can provision and configure a Container Engine Cluster.


### **STEP 1**: Log in to your OCI Console

- Please refer to the Cloud Account that has been provided by your workshop instructor or your administrator. _Please note your account details will be provided separately and are not available on these pages._

- Once you receive the **Oracle Cloud Account**, make note of your **Username, Password, Cloud Tenant Name and Console URL**.

- Open your browser (Safari, Firefox or Chrome) and go to your Oracle Cloud Infrastrcuture Console URL. The OCI Console URL is:

   [https://console.us-ashburn-1.oraclecloud.com/#/a/](https://console.us-ashburn-1.oraclecloud.com/#/a/)

   _This URL may be different to the one your are provided with. Use the provided URL if it is different to the above._

   _You can also access the OCI Console URL from your Cloud Dashboard._

- Click on **Change tenant** button if you are not presented with **Cloud Tenant** input field.

  ![](images/54.png)

- Enter your **Cloud Tenant Name** in the input field and click the **Continue** button. This is supplied by your workshop instructor or administrator earlier.

  ![](images/55.png)


- If you are using an Oracle GSE account, you will be using the default **api.user** to do all provisioning work. Otherwise, you will use your own local user login provided by your Cloud administrator. By default, the local user such as **api.user** does not have the administration rights to create the Kubernetes cluster in OCI. Therefore you must login using the Single Sign-On (SSO) identity with administrative rights and assign the local user (**api.user**) to the admin group. Then log back into the local user account (**api.user**) for the rest of the lab.

- Click **Continue** under the SSO Identity Provider box

- Enter **cloud.admin**  or your Cloud username and its **password** in the input fields and click **Sign In**

  ![](images/56.png)

- In the Console, open the navigation menu. Under **Identity**, click **Groups**. A list of groups is displayed.

- Click on the **Administrators** group

  ![](images/57.png)

- Click **Add User to Group**

- Select **api.user** or your local username from the drop down list and click **Add**

  The **api.user** is now added to the **Administrators** group and you should see something similar to below.

  ![](images/58.png)

- You can now **Sign Out** of the OCI Console.

- Now sign in again using the **api.user** or your local username for Oracle Cloud Infrastructure on the right side of the login page.

- Click **Sign In**

  ![](images/59.png)

- Ensure you're signed in as **api.user** or your local user for the rest of the lab

- You are now ready to create policies


### **STEP 2**: Create Policy for Container Engine

To create and manage clusters in your tenancy, Container Engine must have access to all resources in the tenancy. To give Container Engine the necessary access, create a policy for the service as follows:

- In the Console, click **Identity**, and then click **Policies**. A list of the policies in the compartment you're viewing is displayed.

- Select the tenancy's **root** compartment from the list on the left

- Click **Create Policy**

- Enter the following:
  - **Name:** `oke-service`
  - **Description:** `allow OKE to manage all-resources in tenancy`
  - **Policy Versioning:** Select **Keep Policy Current**
  - **Statement:** The following policy statement:
  `allow service OKE to manage all-resources in tenancy`

- Leave the rest to default

- Click **Create**

  ![](images/21.png)

### **STEP 3**: Configuring Network Resources

You must create a VCN for your cluster and it must include the following:

  - The VCN must have a CIDR block defined that is large enough for at least five subnets, in order to support the number of hosts and load balancers a cluster will have
  - The VCN must have an internet gateway defined
  - The VCN must have a route table defined that has a route rule specifying the internet gateway as the target for the destination CIDR block
  - The VCN must have five subnets defined, three subnets in which to deploy worker nodes and two subnets to host load balancers.

### **STEP 3.1**: VCN Configuration

- In the Console, click **Networking**, and then click **Virtual Cloud Network**

- Select your the tenancy's **Demo** compartment from the compartment list on the left.

- Click **Create Virtual Cloud Network**

- Enter the following:
  - **Name:** `oke-cluster`
  - **CIDR Block:** `10.0.0.0/16`
  - **DNS Resolution:** Check box to **USE DNS HOSTNAMES IN THIS VCN**

- Leave the rest to default (Compartment defaults to **Demo** for GSE env)

  ![](images/22.png)

- Click **Create Virtual Cloud Network**



### **STEP 3.2**: Internet Gateway Configuration

The VCN must have an internet gateway. The internet gateway must be specified as the target for the destination CIDR block 0.0.0.0/0 in a route rule in a route table.

Once the **oke-cluster** VCN is created

- Click on the **oke-cluster** VCN to enter the details page

- Select **Internet Gateways** from the list on the left and click on **Create Internet Gateway**

- Enter the following:
  - **Name:** `oke-gateway-0`

- Leave the rest to default (Compartment defaults to **Demo**)

- Click **Create Internet Gateway**

  ![](images/23.png)


### **STEP 3.3**: Route Table Configuration:

The VCN must have a route table. The route table must have a route rule that specifies an internet gateway as the target for the destination CIDR block 0.0.0.0/0.

- Select **Route Tables** from the list on the left

  ![](images/26.png)

  A `Default Route Table for oke-cluster` should have been created for you, similar to below.

  ![](images/27.png)

If a default Route Table has not been created for you, then create a new Route Table.

- Click **Create Route Table**

- Enter the following:
  - **Name:** `routetable-0`
  - **Destination CIDR block:** `0.0.0.0/0`
  - **Target Type:** `Internet Gateway`
  - **Target Internet Gateway:** `oke-gateway-0`

- Leave the rest to default (Compartment defaults to **Demo** compartment)

- Click **Create Route Table**

  ![](images/24.png)

However, if a default Route Table has been created, then you only need to add a new rule to the Route Table.

- Click on the **Default Route Table for oke-cluster** (default generated name) Route Table to enter the details page

- Click on **Edit Route Rules**

- Enter the following:
  - **Target Type:** `Internet Gateway`
  - **Destination CIDR block:** `0.0.0.0/0`
  - **Compartment:** `Demo`
  - **Target Internet Gateway:** `oke-gateway-0`

  ![](images/25.png)

  - Click **Save**


### **STEP 3.4**: DHCP Options Configuration

The VCN must have a DHCP Options configured. The default value for DNS Type of Internet and VCN Resolver is acceptable.

- In the VCN `oke-cluster` details page, select **DHCP Options** from the list on the left

  A `Default DHCP Options for oke-cluster` should have been created for you

- Click on **Default DHCP Options for oke-cluster** DHCP option to see the detail and you should see something similar to below.

  ![](images/29.png)


### **STEP 3.5**: Security List Configuration

The VCN must have security lists defined for the Worker Node Subnets and the Load Balancer Subnets. Two security lists need to be created (in addition to the default security list) to control access to and from the worker node subnets and load balancer subnets. The two security lists are named **oke-workers** and **oke-loadbalancers** respectively.

Create two additional security lists to the default `Default Security List for oke-cluster`
  - **Security List Name:** `oke-workers`
  - **Security List Name:** `oke-loadbalancers`

There are two types of rules, Ingress and Egress, for both Workers and Load Balancer security lists. There 12 rules in total for the Worker Node Subnet and two rules in total for the Load Balancer Subnet.

Let's create the security lists and rules.

- In the VCN `oke-cluster` details page, select **Security Lists** from the list on the left

  There should be one default security list `Default Security List for oke-cluster` similar to below.

  ![](images/30.png)

### **STEP 3.5.1**: Create Security List for Worker Node Subnets

- Click **Create Security Lists**

- Enter the following for first Ingress rule for **oke-worker** security list name:
  - **Security List Name:** `oke-workers`
  - **Stateless:** `Yes`
  - **Source CIDR:** `10.0.10.0/24`
  - **IP Protocol:** `ALL`


- Click **Add Rule**

  ![](images/31.png)

- Repeat the above step for the remaining Ingress rules in the table below. We have just completed rule number 1, please continue from rule number 2.

  ![](images/32.png)

- Before clicking on **Create Security List** button to complete, you need to enter the Ergress rules as well. There are 4 Egress rules in total.

- Enter the Engress rules following the table below:

  ![](images/33.png)

- Click on **Create Security List** button to complete

**NOTE**: You have completed the **oke-worker** security list for the Worker Node Subnet.


### **STEP 3.5.2**: Create Security List for Load Balancer Subnets

- Repeat Step 3.5.1 for the Load Balancer subnet with the Security List Name of **oke-loadbalancers** following the rules below:

  ![](images/34.png)


- Click on **Create Security List** button to complete

**NOTE**: You have completed the rules for the **oke-loadbalancers** security list.

  You should now have three security lists similar to below:

  ![](images/35.png)




### **STEP 3.6**: Subnet Configuration

We usually require five subnets in the VCN to create and deploy clusters in a highly available configuration. The following configuration assumes you will be deploying across all three Availability Domains.

- Three subnets in which to deploy worker nodes. Each worker node subnet must be in a different availability domain. The worker node subnets must have different security lists to the load balancer subnets.

- Two subnets to host load balancers. Each load balancer subnet must be in a different availability domain. The load balancer subnets must have different security lists to the worker node subnets.


- Still in the VCN page, select **Subnets** from the list on the left

- Click **Create Subnet**

- Enter the following for the first subnet:
  - **Name:** `oke-workers-1`
  - **Availability Domain:** `????:US-ASHBURN-AD-1`
  - **CIDR Block:** `10.0.10.0/24`
  - **Route Table:** `Default Route Table for oke-cluster`
  - **Public Subnet:** `Allow public IP addresses for instances in this Subnet`
  - **DNS Resolution:** `Use DNS Hostnames In This Subnet`
  - **DHCP Options:** `Default DHCP Options for oke-cluster`
  - **Security Lists:** `oke-workers`

  You should have something similar to below:

  ![](images/36.png)

- Click **Create**

- Repeat the above for the remaining two worker subnets **oke-workers-2** and **oke-workers-3** as below:

  **oke-workers-2**

  - **Name:** `oke-workers-2`
  - **Availability Domain:** `????:US-ASHBURN-AD-2`·
  - **CIDR Block:** `10.0.11.0/24`
  - **Route Table:** `Default Route Table for oke-cluster`
  - **Public Subnet:** `Allow public IP addresses for instances in this Subnet`
  - **DNS Resolution:** `Use DNS Hostnames In This Subnet`
  - **DHCP Options:** `Default DHCP Options for oke-cluster`
  - **Security Lists:** `oke-workers`

  **oke-workers-3**

  - **Name:** `oke-workers-3`
  - **Availability Domain:** `????:US-ASHBURN-AD-3`
  - **CIDR Block:** `10.0.12.0/24`
  - **Route Table:** `Default Route Table for oke-cluster`
  - **Public Subnet:** `Allow public IP addresses for instances in this Subnet`
  - **DNS Resolution:** `Use DNS Hostnames In This Subnet`
  - **DHCP Options:** `Default DHCP Options for oke-cluster`
  - **Security Lists:** `oke-workers`


- Repeat the above for the two load balancer subnets **oke-LB-1** and **oke-LB-2** as below:

  **oke-LB-1**

  - **Name:** `oke-LB-1`
  - **Availability Domain:** `????:US-ASHBURN-AD-1`
  - **CIDR Block:** `10.0.20.0/24`
  - **Route Table:** `Default Route Table for oke-cluster`
  - **Public Subnet:** `Allow public IP addresses for instances in this Subnet`
  - **DNS Resolution:** `Use DNS Hostnames In This Subnet`
  - **DHCP Options:** `Default DHCP Options for oke-cluster`
  - **Security Lists:** `oke-loadbalancers`

  **oke-LB-2**

  - **Name:** `oke-LB-2`
  - **Availability Domain:** `????:US-ASHBURN-AD-2`
  - **CIDR Block:** `10.0.21.0/24`
  - **Route Table:** `Default Route Table for oke-cluster`
  - **Public Subnet:** `Allow public IP addresses for instances in this Subnet`
  - **DNS Resolution:** `Use DNS Hostnames In This Subnet`
  - **DHCP Options:** `Default DHCP Options for oke-cluster`
  - **Security Lists:** `oke-loadbalancers`


**NOTE**: You have completed the five subnets. We are ready to create a Kubernetes cluster.



### **STEP 4**: Create a Kubernetes Cluster


- In the Console, click **Developer Services -> Containers Cluster (OKE)**, choose the **Demo** compartment, and then click **Clusters**

- Click **Create Cluster**

- Enter the following configuration details for the new cluster:
  - **Name:** `Demo`
  - **Version:** `v1.9.7` or the latest release
  - **VCN:** `oke-cluster`
  - **Kubernetes Service LB Subnets:** `oke-LB-1`, `oke-LB-2`
  - **Kubernetes Dashboard Enabled:** `Checked`
  - **Tiller (Helm) Enabled:** `Checked`

  Leave the rest of the fields to default and you should have something similar to below:

  ![](images/39.png)

You can either Click **Create** now and create your node pools later OR add the node pools now. Let's add the node pools now to save a step.

- Click **Add Node Pool**

- Enter the following configuration details for the new node pool:
  - **Name:** `demo`
  - **Version:** `v1.9.7` or the latest release
  - **Image:** `Oracle-Linux-7.4`
  - **Shape:** `VM.Standard2.1`
  - **Subnets:** `oke-workers-1`, `oke-workers-2`, `oke-workers-3`
  - **Quantity Per Subnet:** `1`

  This will create three worker nodes, one in each Availability Domain. If you do not require three worker nodes, then just specify the subnet you one to create one in. For example, if only one worker node in AD1, then specify the `oke-workers-1` in the Subnets field.

  Leave the rest of the fields to default and you should have something similar to below:

  ![](images/40.png)

- Click **Create** at the bottom of the form.


The Kubernetes cluster will take a few minutes to create and will be ready for use after it finishes.



### **STEP 5**: Downloading a kubeconfig File to Enable Cluster Access

When you create a cluster, Container Engine creates a Kubernetes configuration file for the cluster called `kubeconfig`. The `kubeconfig` file provides the necessary details to access the cluster using **kubectl** and the Kubernetes Dashboard.

You must download the `kubeconfig` file and set an environment variable to define its location. Having completed the steps, you can start using **kubectl** and the Kubernetes Dashboard to manage the cluster.

**NOTE**: For the current release of Oracle Container Engine, you must install the Oracle Cloud Infrastructure CLI tool in order to download the `kubeconfig` file. In future releases of the OKE, you will be able to download the `kubeconfig` file directly from within the OCI Console via a button action.


### **STEP 5.1**: Generate an API Signing Key Pair

We need to first create a key pair for the OCI CLI to authenticate against your OCI environment where the Container Cluster is running. This will allow the OCI CLI to download the `kubeconfig` file. The public key will be uploaded to OCI user account (**api.user**) via the OCI Console and the OCI CLI will use the private key to authenticate against it.

We will use OpenSSL commands to generate the key pair in the required PEM format.

- If you haven't already, create a .oci directory to store the credentials:

  `mkdir ~/.oci`

- Generate the private key with one of the following commands:

  `openssl genrsa -out ~/.oci/oci_api_key.pem -aes128 2048`

- Ensure that only you can read the private key file:

  `chmod go-rwx ~/.oci/oci_api_key.pem`

- Generate the public key:

  `openssl rsa -pubout -in ~/.oci/oci_api_key.pem -out ~/.oci/oci_api_key_public.pem`

- Copy the contents of the public key to the clipboard using pbcopy, xclip or a similar tool (you'll need to paste the value into the Console later). For example:

  `cat ~/.oci/oci_api_key_public.pem | pbcopy`

- Get the key's fingerprint with the following OpenSSL command:

  `openssl rsa -pubout -outform DER -in ~/.oci/oci_api_key.pem | openssl md5 -c`

The fingerprint will look something similar to the following:

  `12:34:56:78:90:ab:cd:ef:12:34:56:78:90:ab:cd:ef`

This can be used to compared with the generated fingerprint after uploading the public key in the OCI Console. The two fingerprints should be the same.



### **STEP 5.2**: Upload the Public Key of the API Signing Key Pair

You can now upload the PEM public key in the OCI Console.

- In the Console, click **api.user**, and then click **User Settings**. The user details page is now shown.

- Click **Add Public Key**

- Paste the contents of the PEM public key in the dialog box and click **Add**

  You should see something similar to below with the key's fingerprint under the API Keys. This fingerprint should be same as the fingerprint generated by the OpenSSL command in the previous step.

  ![](images/41.png)


### **STEP 5.3**: Installing the Oracle Cloud Infrastructure CLI

The command line interface (CLI) is a tool that enables you to work with Oracle Cloud Infrastructure objects and services. The CLI provides much the same functionality as the Console and includes additional advanced commands.

There are different installation options and steps to install the CLI and required software depending on your platform. It is not possible to cover all the options on this page. Please refer to [Installing the CLI Link](https://docs.us-phoenix-1.oraclecloud.com/Content/API/SDKDocs/cliinstall.htm) for detail on the full installation options and platforms.

**NOTE**: The following steps below are specifically for Linux/MacOS.

To install and use the CLI, you must have:
  - An Oracle Cloud Infrastructure account (**api.user**)
  - A user created in that account, in a group with a policy that grants the desired permissions.
  - A keypair used for signing API requests, with the public key uploaded to Oracle.
  - Python version 2.7.5 or 3.5 or later

There are two ways for installing the CLI, Automatic or Manual. We recommend the Automatic way as this uses the CLI Installer to install all the dependencies.

To install the CLI on any computer with Bash:

  1. Open a terminal or shell

  2. Enter the following command to run the installer script

  curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh

  3. Respond to the installer's prompts



### **STEP 5.4**: Configuring the Oracle Cloud Infrastructure CLI

Before using the CLI, you have to create a config file that contains the required credentials for working with Oracle Cloud Infrastructure. You can create this file using a setup dialog or manually, using a text editor. It is recommended to use the setup dialog.

- Open a shell and run the `oci setup config` command

  The command prompts you for the information required for the config file and the API public/private keys.

- When prompted for the API public/private keys, you can specify the keys you generated previously.


### **STEP 5.5**: Download the kubeconfig.sh File

- In the Console, open the navigation menu. Click **Containers**

- Choose the **Demo** compartment, and then click **Clusters**

- On the **Cluster** List page, click the **Demo** cluster you created. The Cluster page shows details of the cluster similar to below:

  ![](images/42.png)

- Click the **Access Kubeconfig** button to display the **How to Access Kubeconfig dialog box**

  ![](images/43.png)



- Run the commands shown in the dialog box in a terminal or shell.

  `mkdir -p $HOME/.kube`
  `oci ce cluster create-kubeconfig --cluster-id ocid1.cluster.oc1.iad.aaaaaaaaaeydeyrthaytonjwmfstcmbwhbsdcnbqmzrgimztgcsdqztdhbrd --file $HOME/.kube/config`


- For convenience, the command in the How to **Access Kubeconfig** dialog box already includes the cluster's OCID. You can simply copy and paste that command intoo your terminal or shell.


### **STEP 5.6**: Set the KUBECONFIG environment variable

- In a terminal window, set the **KUBECONFIG** environment variable to the name and location of the `kubeconfig` file. For example, on Linux or MacOS:

  `$ export KUBECONFIG=~/kubeconfig`

- Verify that **kubectl** is available and that it can connect to the cluster

  `$ kubectl get nodes`

  Information about the nodes in the cluster should be shown.

You can now use `kubectl` and the Kubernetes Dashboard to perform operations on the cluster.


### **STEP 6**: Starting The Kubernetes Dashboard

Kubernetes Dashboard is a web-based user interface that you can use as an alternative to the Kubernetes kubectl command line tool to:

- Deploy containerized applications to a Kubernetes cluster
- Troubleshoot your containerized applications

You use the Kubernetes Dashboard to get an overview of applications running on a cluster, as well as to create or modify individual Kubernetes resources. The Kubernetes Dashboard also reports the status of Kubernetes resources in the cluster, and any errors that have occurred.

In contrast to the Kubernetes Dashboard, Container Engine enables you to create and delete Kubernetes clusters and node pools, and to manage the associated compute, network, and storage resources.

Before you can use the Kubernetes Dashboard, you need to start a Kubernetes proxy locally to connect you to the Kubernetes master.

- You can find the command to start the proxy and the URL to the Kubernetes Dashboard under the **Getting Started** option on the left of your cluster page.

  ![](images/60.png)

To start the Kubernetes Dashboard:

- In a terminal window where you have exported the `KUBECONFIG` environment variable, enter
  `kubectl proxy` to start the proxy server to enable Kubernetes Dashboard access.

- With the proxy server running, open a browser and go to:
  http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy to enter the Kubernetes Dashboard.

  ![](images/53.png)



## **STEP 7 **: Deploying to a Cluster

The Kubernetes cluster setup is complete and you are now able to deploy your application to this cluster. This cluster is can be reached by its cluster address. You can find this address either through the OCI console or in the `kubeconfig` file.

- Open the `kubeconfig` file and look for the line with the **server** tag. For example:

  `server: https://cygiyzvmi4w.us-ashburn-1.clusters.oci.oraclecloud.com:6443`

  This is the cluster address you should use for deployment, such as the one specified for `KUBERNETES_MASTER` in **Wercker Environment**.

Alternatively:

- In the Console, open the navigation menu. Click **Containers**

- Choose the **Demo** compartment, and then click **Clusters**

- On the **Cluster** List page, click the **Demo** cluster you created.

- Under the Cluster Details, you will find the Kubernetes Cluster address similar to below:

  ![](images/52.png)

  This should be the same address as the one found in `kubeconfig`.

  You are ready to config Wercker to deploy to this cluster.


- Great! We've got Kubernetes installed and accessible -- now we're ready to get our microservice deployed to the cluster.
