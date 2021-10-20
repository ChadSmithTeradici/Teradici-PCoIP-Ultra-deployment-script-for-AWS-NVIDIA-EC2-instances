---
title: Install PCoIP Ultra via a script in AWS
description: Steps to utilize a deployment script for Teradici PCoIP on AWS/Nvidia Instances (G3, G4dn, G5dn). This is outside of an AWS marketplace offering.
author: chad-m-smith
tags: Teradici, AWS, Nvidia, EC2
date_published: 2021-10-20
---

Chad Smith | Technical Alliance Architect at Teradici | HP

<p style="background-color:#CAFACA;"><i>Contributed by Teradici employees.</i></p>

This guide shows you how to install Teradici PCoIP agent on a Nvidia powered Instance running in AWS. Also this guide is intended for customers that have Teradici annual subcription and are interested in transfering licensed seats to a AWS EC2 instance(s). There is an alternative option for a AWS marketplace hourly subscription with pre-packaged AMI for [Windows 2019](https://aws.amazon.com/marketplace/pp/prodview-boeg6hiewus3o?sr=0-1&ref_=beagle&applicationId=AWSMPContessa) and [CentOS 7](https://aws.amazon.com/marketplace/pp/prodview-yjdn554yaqvem?sr=0-2&ref_=beagle&applicationId=AWSMPContessa). AWS marketplace offering is NOT apart of this deployment guide. 

EC2 instances are available for purchase through On Demand and Savings Plans pricing models. Billing for EC2 Mac instances is per second with a 1hr-hour minimum allocation period to comply with the Mircosft Software License Agreement for windows. You can launch an EC2 Instanes and be up and running within minutes. At the end of the 1-hour minimum allocation period, the host can be released at any time without further commitment. 

More Information on EC2 Instance can be found [here](https://aws.amazon.com/ec2/pricing/on-demand/).

## Objectives

+ Allocate a AWS EC2 Nvidia powered Instance from AWS Console.
+ Drop in deployment script based on Instance OS
+ Configure Security Groups to allows access to instance (SSH,RDP & PCoIP ports).
+ Connect to EC2 Instance via PCoIP client

## Costs

This tutorial uses billable components of AWS Cloud and assumes Teradici subscription, including the following:

+   [Teradici PCoIP](https://connect.teradici.com/contact-us), Teradici PCoIP subscriptions
+   [AWS Nvidia EC2 Instance](https://aws.amazon.com/nvidia/), including vCPUs, memory, disk, and GPUs
+   [Internet egress and transfer costs](https://aws.amazon.com/blogs/architecture/overview-of-data-transfer-costs-for-common-architectures/), for PCoIP and other applications communications

Use the [AWS pricing calculator](https://calculator.aws/#/) to generate a cost estimate based on your projected usage.

## Before you begin

In this section, you set up some basic resources that the tutorial depends on.

1. Instructions in this guide assume that you have a [AWS account](https://aws.amazon.com/free/) 

1. Ensure you have [Service Quotas](https://console.aws.amazon.com/servicequotas) for **'All G and VT Spot Instance Requests'**.

1. Familiarize yourself with [AWS network](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Networking.html) topology and best practices.

1. Obtain a [Teradici PCoIP registation](https://connect.teradici.com/contact-us) code that has at least one un-assigned seat available.

## Set up the virtual workstation

In this section, you create and configure a virtual workstation, including setting up networking and installing utilities. 

### Procure the EC2 Nvidia Instance

In this section, you procure a G4dn/G5dn type dedicated host in your region

1. Select a AWS region that has [EC2 G4dn Instances available](https://www.instance-pricing.com/provider=aws-ec2/instance=g4dn.4xlarge/) with a understanding of hourly consumption rate. 

1.  Launch a G4dn instance, On the [EC2 Dashboard](https://console.aws.amazon.com/ec2), choose **Launch Instance**.

1. On the **Choose AMI** page, select the [Windows 2019 Base](https://aws.amazon.com/marketplace/pp/prodview-bd6o47htpbnoe?ref=cns_srchrow) or [Cent0S7](https://aws.amazon.com/marketplace/pp/prodview-qkzypm3vjr45g?ref=cns_srchrow) AMI(s) based on desired OS then press **Select** button.

1. On the **Choose Instance Type** page, keep the default selection of **G4dn Instance familiy types** and choose **Next: Configure Instance Details**.

    ![image](https://github.com/ChadSmithTeradici/Teradici-PCoIP-deployment_script-for-AWS-NVIDIA-Instances/blob/main/images/AWS-G4dn-Fam.jpg)

1. On the **Configure Instance Details** page, for **Host**, choose the **Dedicated Host** you allocated earlier. For the remaining configuration details, make any selections you prefer. Then, choose **Next: Add Storage**.

1. On the **Add Storage** page, choose the Size (GiB) cell and increase the volume based on your requirements. Then, choose **Next: Add Tags**.

1. On the **Add Tags page**, optionally add any Key:Value tags to your instance. Then, choose **Next: Configure Security Group**.

1. On the Configure Security Group page, make the following selections:

    + For **Assign a security group**, choose **Create a new security group**.
    + For **Security group name**, type a descriptive name, such as *pcoip ssh into mac1.metal*.
    + For **Description**, optionally add a description.
    + For **Type**, choose **SSH**
    + For **Source**, choose **My IP**
    + Select **Add rule**
    + For **Type**, choose **Custom TCP Rule**
    + For **Port Range** choose **HTTPS**
    + For **Source**, choose **0.0.0.0/0**
    + Select **Add rule**
    + For **Type**, choose **Custom TCP Rule**
    + For **Port Range** choose **4172**
    + For **Source**, choose **0.0.0.0/0**
    + Select **Add rule**
    + For **Type**, choose **Custom UDP Rule**
    + For **Port Range** choose **4172**
    + For **Source**, choose **0.0.0.0/0**
    + Select **Add rule**
    + For **Type**, choose **Custom TCP Rule**
    + For **Port Range** choose **5900**
    + For **Source**, choose **My IP**
    +  Select **Add rule**
    + For **Type**, choose **Custom TCP Rule**
    + For **Port Range** choose **5800**
    + For **Source**, choose **My IP**

    Then, choose **Review** and **Launch**.

1. On the **Review page**, review your selections and verify that the **Host ID** matches the Dedicated Host you created earlier. Then, choose **Launch**.

1. On the **Select an existing key pair or create a new key pair** dialog, verify your existing key pair (if you do not have a key pair, select the option to create a new key pair). Then, select the acknowlegement check box and choose **Launch Instances**.

1. On the **Instances** page, wait for the **Status Check** column of your instance to show 2/2 checks passed before continuing.

## Set up the connection to your EC2 Mac Instance

