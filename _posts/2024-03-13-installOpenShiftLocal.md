---
layout: post
title:  "Install OpenShift Local"
author: Sean Boyd
---

## Intro

I've installed and reinstalled OpenShift Local multiple times for testing on both my laptop and my desktop PC. This blog describes the condensed steps I followed to install OpenShift Local on my desktop PC. For a detailed guide refer to the references section.

## References

Primary Red Hat landing page:

[OpenShift overview](https://developers.redhat.com/products/openshift-local/overview)

Download page for OpenShift Local. This includes the pull secret and a link to the setup guide:

[Download OpenShift Local](https://console.redhat.com/openshift/create/local)

Setup guide. This is what I followed. This blog has extracted the bare-bones from this guide to install and configure OpenShift Local.

[OpenShift Local installation guide](https://access.redhat.com/documentation/en-us/red_hat_openshift_local)

## Minimum requirements

According to the OpenShift Local guide the following minimum requirements must be met:

- 4 physical CPU cores
- 9 GB of free memory
- 35 GB of storage space

You cannot install OpenShift Local using Windows Home Edition.

## Environment

| Category | Details  |
| :--- | :--- |
| Desktop PC  | HP Omen, 12th Gen Intel Core i7-12700K, 3600 MHz, 12 cores, 20 logical processors |
| Primary disk  | NVMe WD WD_BLACK Gen4 1TB disk |

The sizings I specified for the configuration is higher than the defaults for multiple reasons:

- I ran into issues with the default sizes not being high enough
- With the desktop PC specs I am able to set higher settings than the defaults which appear to be set as a minimum to run OpenShift Local on laptops
- I want to enable cluster monitoring and according to the OpenShift Local installation guide this requires a minumum of 14GB memory

## Download OpenShift Local

The first step is to download OpenShift Local. In my case I downloaded the Windows x86_64 installer.

[https://console.redhat.com/openshift/create/local](https://console.redhat.com/openshift/create/local)

This requires you to create and log into a Red Hat account.

Click the download button to download the file "crc-windows-installer.zip"

Unzip this file, run the installer and follow the prompts.

Take note of the following comment from the OpenShift Local installation guide: 

*"On Microsoft Windows, you must install Red Hat OpenShift Local to your local C:\ drive. You cannot run Red Hat OpenShift Local from a network drive."*

## Usage data collection

I chose to disable usage data collection

```cmd
crc config set consent-telemetry no
```

## Change selected defaults

From my general usage it is not necessary to change the defaults. I have been using OpenShift Local for some time without any issues.

Recently, however, I started to face issues with ephemeral storage and DiskPressure. So I decided to increase selected defaults.

These defaults should be set before issuing the "crc setup".

Update the number of CPUS from 4 (default) to 6. I'm not sure whether this is really needed but I decided to change anyway.

```cmd
crc config set cpus 6
```

Update the memory from 9 GB (default) to 25 GB. As I plan on enabling the monitoring that requires a minimum of 14 GB I chose a much higher value. To check the usage you can issue "crc status" - after installation I can see this going much higher than the default due to the components I am installing.

```cmd
crc config set memory 25600
```

I also used this as a chance to increase the disk size from 31 GB (default) to a much higher value. I chose 100 GB for no reason other than it's a much larger number.

```cmd
crc config set disk-size 100
```

Lastly I have decided to enable cluster monitoring. The default is off and the documentation states a minimum of 14 GB memory is required if this feature is enabled.

```cmd
crc config set enable-cluster-monitoring true
```

## Complete the setup

To complete the setup issue the command.

```cmd
crc setup
```

Once the setup is complete, start OpenShift Local.

```cmd
crc start
```

You should now be done and ready to use OpenShift Local!

## Some useful commands

```cmd
crc config
crc config get cpus
crc config get memory
crc config get disk-size
crc status
```

## Output from "crc setup"

````
INFO Using bundle path C:\Users\sean\.crc\cache\crc_hyperv_4.14.12_amd64.crcbundle
INFO Checking minimum RAM requirements
INFO Checking if current user is in crc-users and Hyper-V admins group
INFO Checking if CRC bundle is extracted in '$HOME/.crc'
INFO Checking if C:\Users\sean\.crc\cache\crc_hyperv_4.14.12_amd64.crcbundle exists
INFO Getting bundle for the CRC executable
INFO Downloading bundle: C:\Users\sean\.crc\cache\crc_hyperv_4.14.12_amd64.crcbundle...
4.33 GiB / 4.33 GiB [----------------------------------------------------------------------------------------------------------------------------------------------------------------------] 100.00% 20.21 MiB/s
INFO Uncompressing C:\Users\sean\.crc\cache\crc_hyperv_4.14.12_amd64.crcbundle
crc.vhdx:  18.99 GiB / 18.99 GiB [---------------------------------------------------------------------------------------------------------------------------------------------------------------------] 100.00%
oc.exe:  117.17 MiB / 117.17 MiB [---------------------------------------------------------------------------------------------------------------------------------------------------------------------] 100.00%
INFO Checking if the win32 background launcher is installed
INFO Checking if the daemon task is installed
INFO Installing the daemon task
INFO Checking if the daemon task is running
INFO Running the daemon task
INFO Checking admin helper service is running
INFO Checking SSH port availability
Your system is correctly setup for using CRC. Use 'crc start' to start the instance
````

## Output from "crc start"

````
INFO Using bundle path C:\Users\sean\.crc\cache\crc_hyperv_4.14.12_amd64.crcbundle
INFO Checking minimum RAM requirements
INFO Checking if running in a shell with administrator rights
INFO Checking Windows release
INFO Checking Windows edition
INFO Checking if Hyper-V is installed and operational
INFO Checking if Hyper-V service is enabled
INFO Checking if crc-users group exists
INFO Checking if current user is in crc-users and Hyper-V admins group
INFO Checking if vsock is correctly configured
INFO Checking if the win32 background launcher is installed
INFO Checking if the daemon task is installed
INFO Checking if the daemon task is running
INFO Checking admin helper service is running
INFO Checking SSH port availability
INFO Loading bundle: crc_hyperv_4.14.12_amd64...
CRC requires a pull secret to download content from Red Hat.
You can copy it from the Pull Secret section of https://console.redhat.com/openshift/create/local.
? Please enter the pull secret **********************************************************************************************************************************************************************************INFO Creating CRC VM for OpenShift 4.14.12...
INFO Generating new SSH key pair...
INFO Generating new password for the kubeadmin user
INFO Starting CRC VM for openshift 4.14.12...
INFO CRC instance is running with IP 127.0.0.1
INFO CRC VM is running
INFO Updating authorized keys...
INFO Resizing /dev/sda4 filesystem
INFO Check internal and public DNS query...
INFO Check DNS query from host...
INFO Verifying validity of the kubelet certificates...
INFO Starting kubelet service
INFO Waiting for kube-apiserver availability... [takes around 2min]
INFO Adding user's pull secret to the cluster...
INFO Updating SSH key to machine config resource...
INFO Waiting until the user's pull secret is written to the instance disk...
INFO Changing the password for the kubeadmin user
INFO Updating cluster ID...
INFO Enabling cluster monitoring operator...
INFO Updating root CA cert to admin-kubeconfig-client-ca configmap...
INFO Starting openshift instance... [waiting for the cluster to stabilize]
INFO 5 operators are progressing: console, image-registry, ingress, network, openshift-controller-manager
INFO Operator openshift-controller-manager is progressing
INFO All operators are available. Ensuring stability...
INFO Operators are stable (2/3)...
INFO Operators are stable (3/3)...
INFO Adding crc-admin and crc-developer contexts to kubeconfig...
Started the OpenShift cluster.

The server is accessible via web console at:
  https://console-openshift-console.apps-crc.testing

Log in as administrator:
  Username: kubeadmin
  Password: xxxxxxxxxxxxxxxxxxxxxxxxx

Log in as user:
  Username: developer
  Password: developer

Use the 'oc' command line interface:
  > @FOR /f "tokens=*" %i IN ('crc oc-env') DO @call %i
  > oc login -u developer https://api.crc.testing:6443
````

## Final comment

As part of my learning, I followed the below OpenLiberty guide. This put my OpenShift Local installation through its paces and is the primary reason I increased the default sizes. With the defaults I ran into problems with ephemeral memory and Disk Pressure.

[OpenLiberty Guide Cloud OpenShift Operator](https://openliberty.io/guides/cloud-openshift-operator.html)
