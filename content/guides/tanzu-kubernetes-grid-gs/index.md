---
title: "Getting Started with VMware Tanzu Kubernetes Grid"
description: This guide will walk you through setting up a basic VMware Tanzu Kubernetes Grid workload cluster. This cluster will be useful for setting up an inner loop development workflow using Tanzu tooling. 
date: 2021-12-10
lastmod: 2021-12-10
level1: Building Kubernetes Runtime
level2: Application Platform on Kubernetes
tags:
- Tanzu
- Tanzu Kubernetes Grid
# Author(s)
team:
- Tony Vetter
---

{{% callout %}} 
This guide was written for Tanzu Kubernetes Grid `v1.4`. It should also be noted that Tanzu Kubernetes Grid is not a free product. This guide assumes that you have a licensing agreement in place with VMware which includes this Tanzu Kubernetes Grid entitlement. This guide also assumes a MacOS v12.0. This guide will be updated regularly, but updates might not be timed exactly to new releases of these products or platforms. You may need to modify commands. 
{{% /callout %}}

VMware Tanzu Kubernetes Grid is a production Kubernetes platform for organizations to give their teams a unified experience regardless of where the cluster is running. Built for multi-cloud deployments, Tanzu Kubernetes Grid seeks to simplify cluster lifecycle management, as well as application and dependency deployments. 

As a developer, Tanzu Kubernetes Grid represents an easy way for you to get up and running with Kubernetes. And not just for small, "dev and test" environment. But environments that can scale from small, individual-use clusters all the way up to production multicloud environments. And everything in between. All while providing a uniform experience and environment for application deployment workflows. 

In this guide, you will deploy Tanzu Kubernetes Grid. You will deploy a Management Cluster, as well as a singe Workload CLuster. This workload cluster will be used in later guides as you build toward developing with Tanzu tools. 

## Prerequisites

* **You have read through and completed the previous guides in this series.**
  * [Getting Started with the VMware Tanzu Network](/guides/tanzu-network-gs) - This guide walks you through setting up your account on the Tanzu Network, as well as installing the `pivnet` CLI tool.
  * [Getting Started with the tanzu CLI tool](/guides/tanzu-cli-gs) - This guide walks you through downloading, installing, and using the `tanzu` CLI tool. 
* [Install `kubectl`](https://kubernetes.io/docs/tasks/tools/) - This tool will be used for managing the various Kubernetes clusters you will be creating in this guide. 
* [An Amazon Web Services account](https://aws.amazon.com/free) - Tanzu Kubernetes Grid supports different underlying platforms on which it runs. For example Azure, VMware vSphere, and Amazon Web Services. But for this guide I have opted for Amazon Web Services. You will need an account with access to EC2 and a couple other common services. 
* [Install the `awscli`](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) - In addition to the base install, ensure that `aws configure` has been run successfully. The Tanzu Kubernetes Grid cluster installer will read from the information provided in the `aws` CLI for authorization and default settings.
* [Create an EC2 SSH key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair) - You will need to create a key pair using one of the supported methods, and record the name of the key pair you want to use.
* **Optional**: [Install Visual Studio Code](https://code.visualstudio.com/download) - You will need a text editor of some kind. Commands provided in this guide will open Visual Studio Code, but it is not required that you use this. 
* **Probably more**(TODO)

## Creating a Management Cluster 

A management cluster is the first element you will need to deploy to create your Tanzu Kubernetes Grid environment. A management cluster runs [Cluster API](https://github.com/kubernetes-sigs/cluster-api), which is what will not only create your workload clusters, but also configure, update, and perform other lifecycle tasks. 

It may seem like a lot to deploy a separate management cluster when the eventual goal is simply an inner-loop development environment. It is worth noting that this process is only in place in this guide for learning purposes. In a production environment, this would likely not be necessary. 

Your team, or organization would likely already have a centralized management cluster. And in this case all you would need to do would be to [create a workload cluster](). Or have one created for you that you could simply log into. 

But since this guide is for learning purposes, you will be deploying the whole stack. 

1. Start the process by logging into the management cluster creation UI. There is a way to do this over CLI as well, but starting with the UI will build for you the YAML template you need to make this a repeatable deployment. 
    
    ```sh
    tanzu management-cluster create --ui
    ```

2. From here, a web UI will be launched automatically. Select **Deploy** under Amazon Web Services.
    
    ![The management cluster creation UI](images/screenshots/image1.png "Click deploy under Amazon Web Services.")

3. **IaaS Provider**: Fill out the **AWS Credentials Profile** you want to use to authenticate to AWS, and the **Region** you want your management cluster deployed to. Then click **Connect**.
    
    ![The IaaS Provider screen](images/screenshots/image2.png "Fill out the credentials profile and the region.")
    
    If the **Connect** button switches to **Connected**, then your credentials were authenticated and you can click **Next**.

4. **VPC for AWS**: For your purposes of creating an environment meant for learning, you can leave this section as default. Click **Next**.
    
    ![The VPC for AWS screen](images/screenshots/image3.png "Leave as default and click next.")

5. **Management Cluster Settings**: Fill out the following fields on this screen:
   1. Select **Development** as the cluster type.
   2. Select an EC2 **Instance Type** for the control plane nodes. I am using `m4.large`, but you can use other types if you want.
   3. Set a **Management Cluster Name**. This is optional and can be anything you want. It will be used in the kubectl config file. 
   4. Set the **EC2 Key Pair**. This should have been set up in the [prerequisites](#prerequisites) section of this guide. Just put the reference name here.
   5. Select an **Availability Zone**. Any AZ in your configured region is fine.
   6. Set a **Worker Node Instance Type**. Similar to the control plane instance type, I am using m4.large again, just for simplicity. 
   7. When all of these are set, click **Next**.

    ![The Management Cluster Settings screen](images/screenshots/image4.png "Fill out the fields in red, and click next.")

6. **Netadata**: This section is for adding metadata to your cluster which would be helpful in organizing it against other management and workload clusters. Since this is a very small test environment, this can be left blank. Click **Next**.

    ![The Metadata screen](images/screenshots/image5.png "Leave as default and click next.")

7. **Kubernetes Network**: This section will allow you to customize the management cluster network settings. Again, since this is a small test environment, this can be left as default. Click **Next**.

    ![The Metadata screen](images/screenshots/image5.png "Leave as default and click next.")

8. **Identity Management**: Disable Identity Management Settings. Setting up an identity provider is outside the scope of this guide. For our purposes, this is a reasonably safe setting. However, do not disable this in a production scenario. Click **Next**. 

    ![The Identity Management screen](images/screenshots/image5.png "Disable identity management and click next.")

9. **OS Image**: Select an operating system image type. I am using Amazon AMI, but Ubuntu will work fine too. 

    ![The OS Image screen](images/screenshots/image5.png "Select and OS image type and click next.")

10. **CEIP Agreement**: If you would like to participate in VMware's Customer Experience Improvement Plan, simply click **Next**. Otherwise you can opt out before proceeding. 

    ![The CEIP Agreement screen](images/screenshots/image5.png "Leave as default and click next, or opt out first.")

11. Click **Review Configuration**.

    ![Review screen](images/screenshots/image5.png "Click review configuration.")

12. Review your configuration. At the bottom of the screen, take note of the CLI command and the location of the YAML template. This is the template generated by the configuration UI. To repeat this deployment, you just need to deploy against this template. When you are ready, click **Deploy Management Cluster**.

    ![Confirmation screen](images/screenshots/image5.png "Click deploy management clust")

    There will be logs streaming in the UI, as well as in the terminal window where you originally ran the `create` command. This will take some time. 

13. List your `kubectl` contexts, and connect to your workload cluster.

    ```sh
    kubectl config get-contexts
    ```
    
    ```sh
    kubectl config use-context <context-name>
    ```

## Creating a Workload Cluster

You can now create your workload cluster. Much of the configuration for this cluster will use the same configuration as your workload cluster, unless you over ride certain configuration options. 

A sample configuration file will be provided below. This file will have the minimum configuration options necessary to create your workload cluster. Other configuration options will be copied from your management cluster unless you over ride them in this file. 

1. Download the sample configuration file. 

    ```sh
    git clone https://gist.github.com/29d297d0cc2a5a15607f1f14a2906a87.git 
    ```

2. Open the config file and edit it as necessary. You will need to, at a minimum, modify `AWS_SSH_KEY_NAME`. This is the same SSH key name as you used to create your management cluster in the previous section. Modify other options as desired. Optional and common configuration options to change are highlighted in the file.

    ```sh
    code 29d297d0cc2a5a15607f1f14a2906a87/tkg-1-4-config
    ```

3. Create the workload cluster.

    ```sh
    tanzu cluster create test-workload-cluster -f 29d297d0cc2a5a15607f1f14a2906a87/tkg-1-4-config
    ```

    This will begin the process of creating your workload cluster. This will take a few minutes to complete.

4. Pull the kubeconfig context from the cluster. This command will allow you admin access to the cluster.

    ```sh
    tanzu cluster kubeconfig get test-workload-cluster --admin
    ```

5. List your kubeconfig contexts and use the correct one for your new workload cluster.

    ```sh
    kubectl config get-contexts
    ```

    ```sh
    kubectl config use-context <context-name>
    ```

6. Verify the cluster health by pulling some cluster details.

    ```sh
    tanzu cluster get test-workload-cluster -v 1
    ```

    Example Output:
    ```sh
    NAME                   NAMESPACE  STATUS   CONTROLPLANE  WORKERS  KUBERNETES        ROLES
    test-workload-cluster  default    running  1/1           1/1      v1.22.3+vmware.1  <none>

    Details:

    NAME                                                                      READY  SEVERITY  REASON  SINCE  MESSAGE
    /test-workload-cluster                                                    True                     63m
    ├─ClusterInfrastructure - AWSCluster/test-workload-cluster                True                     65m
    ├─ControlPlane - KubeadmControlPlane/test-workload-cluster-control-plane  True                     63m
    │ └─Machine/test-workload-cluster-control-plane-229pb                     True                     65m
    └─Workers
        └─MachineDeployment/test-workload-cluster-md-0                        True                     61m
            └─Machine/test-workload-cluster-md-0-558f7bf676-xw22s             True                     63m
    ```

    Here you can see various, high-level information about your cluster. Including the Kubernetes version, and the status of your infrastructure. 
    
## Next Steps

There are many other management functions you can familiarize your self with. Try running `tanzu cluster --help` to get an idea of what is possible, and start exploring.

Next, try out installing Tanzu Application Platform on top of your workload cluster as part of your development environment. 
* Getting Started with VMware Tanzu Application Platform `dev-light` Profile (Coming Soon) - This guide will introduce Tanzu Application Platform, and walk you though installing it on top of your workload cluster. This will later be used to create your Tanzu development environment. 