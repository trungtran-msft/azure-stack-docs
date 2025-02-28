---
title: Troubleshoot known issues and errors when installing Azure Kubernetes Service on Azure Stack HCI 
description: Find solutions to known issues and errors when installing Azure Kubernetes Service on Azure Stack HCI 
author: mattbriggs
ms.topic: troubleshooting
ms.date: 02/08/2022
ms.author: mabrigg 
ms.lastreviewed: 02/08/2022
ms.reviewer: abha

---

# Resolve issues and errors during an AKS on Azure Stack HCI installation

This article describes known issues and errors you may encounter when running an installation of AKS on Azure Stack HCI. You can also review known issues with when [upgrading](known-issues-upgrade.md) and using [Windows Admin Center](known-issues-windows-admin-center.md).

## During installation, this error appears: `unable to create appliance VM: cannot create virtual machine: rpc error = unknown desc = Exception occurred. (Generic failure)]`

This error occurs when Azure Stack HCI is out of policy. The connection status on the cluster may show it's connected, but the event log shows the warning message that `Azure Stack HCI's subscription is expired, run Sync-AzureStackHCI to renew the subscription`.

To resolve this error, verify that the cluster is registered with Azure using the `Get-AzureStackHCI` PowerShell cmdlet that's available on your machine. The Windows Admin Center dashboard also shows status information about the cluster's Azure registration.

If the cluster is already registered, then you should view the `LastConnected` field in the output of `Get-AzureStackHCI`. If the field shows it's been more than 30 days, you should attempt to resolve the situation by using the `Sync-AzureStackHCI` cmdlet.

You can also validate whether each node of your cluster has the required license by using the following cmdlet:

```powershell
Get-ClusterNode | % { Get-AzureStackHCISubscriptionStatus -ComputerName $_ }
```

```output
Computer Name Subscription Name           Status   Valid To
------------- -----------------           ------   --------
MS-HCIv2-01   Azure Stack HCI             Active   12/23/2021 12:00:14 AM
MS-HCIv2-01   Windows Server Subscription Inactive

MS-HCIv2-02   Azure Stack HCI             Active   12/23/2021 12:00:14 AM
MS-HCIv2-02   Windows Server Subscription Inactive

MS-HCIv2-03   Azure Stack HCI             Active   12/23/2021 12:00:14 AM
MS-HCIv2-03   Windows Server Subscription Inactive
```

If the issue isn't resolved after running the `Sync-AzureStackHCI` cmdlet, you should reach out to Microsoft support.

## Error: `Install-MOC failed with the error - the process cannot access the file \<path> because it is being used by another process.`

The file cannot be access because it is being used by another process.

You can resolve this issue by restarting your PowerShell session. Close the PowerShell window and try Install-MOC again.


## Error: `Install-Moc failed with error - Exception [CloudAgent is unreachable. MOC CloudAgent might be unreachable for the following reasons]` 

This error may occur when there's an infrastructure misconfiguration. Use the following steps to resolve this error:

1. Check the host DNS server configuration and gateway settings:
   1. Confirm that the DNS server is correctly configured. To check the host's DNS server address, run the following command: 
      ```powershell
      ((Get-NetIPConfiguration).DNSServer | ?{ $_.AddressFamily -ne 23}).ServerAddresses
      ```
   2. To check whether your IP address and gateway configuration are correct, run the command `ipconfig/all`.
   3. Attempt to ping the IP gateway and the DNS server.

2. Check the CloudAgent service to make sure it's running:
   1. Ping the CloudAgent service to ensure it's reachable.
   2. Make sure all nodes can resolve the CloudAgent's DNS by running the following command on each node:
      ```powershell
      Resolve-DnsName <FQDN of cloudagent>
      ```
   3. When the previous step succeeds on the nodes, make sure the nodes can reach the CloudAgent port to verify that a proxy is not trying to block this connection and the port is open. To do this, run the following command on each node:
      ```powershell
      Test-NetConnection <FQDN of cloudagent> -Port <Cloudagent port - default 65000>
      ```
   4. To check if the cluster service is running for a failover cluster, you can also run the following command:
      ```powershell
      Get-ClusterGroup -Name (Get-AksHciConfig).Moc['clusterRoleName']
      ```

## Error:  `[Doc]Install-Moc failed with error - [The object already exists] An error occurred while creating resource 'IPv4 Address xxx.xx.xx.xx' for the clustered role 'xx-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx'

A previously installed feature remains in a failed state and has not been cleared.

You will need to manually clean up the cluster role manually. You can remove the resource from the  fail-over  cluster manager by running the following PowerShell cmdlet:  `Remove-ClusterResource -name <resource name>`.
## Error: `Install-Moc failed with error - Exception [Could not create the failover cluster generic role.]`  

This error indicates that the cloud service's IP address is not a part of the cluster network and doesn't match any of the cluster networks that have the `client and cluster communication` role enabled.

To resolve this issue, run [Get-ClusterNetwork](/powershell/module/failoverclusters/get-clusternetwork?view=windowsserver2019-ps&preserve-view=true) where `Role` equals `ClusterAndClient`. Then, on one of the cluster nodes, select the name, address, and address mask to verify that the IP address provided for the `-cloudServiceIP` parameter of [New-AksHciNetworkSetting](./reference/ps/new-akshcinetworksetting.md) matches one of the displayed networks.

## Error: `The process cannot access the file 'mocstack.cab' because it is being used by another process`

`Install-AksHci` failed with this error because another process is accessing `mocstack.cab`. To resolve this issue, close all open PowerShell windows and then reopen a new PowerShell window.
 
## Error: `An existing connection was forcibly closed by the remote host`

`Install-AksHci` failed with this error because the IP pool ranges provided in the AKS on Azure Stack HCI configuration was off by one in the CIDR, and can cause CloudAgent to crash. For example, if you have subnet 10.0.0.0/21 with an address range 10.0.0.0 - 10.0.7.255, and then you use start address of 10.0.0.1 or an end address of 10.0.7.254, then this would cause CloudAgent to crash. 

To work around this issue, run [New-AksHciNetworkSetting](./reference/ps/new-akshcinetworksetting.md) and use any other valid IP address range for your VIP pool and Kubernetes node pool. Make sure that the values you use are not off by one at the start or end the address range. 

## After deploying AKS on Azure Stack HCI 21H2, rebooting the nodes showed a failed status for billing

After deployment, when rebooting the Azure Stack HCI nodes, the AKS report showed a failed status for billing. To resolve this issue, follow the instructions to [manually rotate the token and restart the KMS plug-in](known-issues-workload-clusters.md#the-api-server-is-not-responsive-after-several-days).

## An `Unable to acquire token` error appears when running Set-AksHciRegistration
This error can occur when you have multiple tenants on your Azure account. Use `$tenantId = (Get-AzContext).Tenant.Id` to set the right tenant. Then, include this tenant as a parameter while running [Set-AksHciRegistration](./reference/ps/set-akshciregistration.md). 

## Cloud agent may fail to successfully start when using path names with spaces in them
When using [Set-AksHciConfig](./reference/ps/set-akshciconfig.md) to specify `-imageDir`, `-workingDir`, `-cloudConfigLocation`, or `-nodeConfigLocation` parameters with a path name that contains a space character, such as `D:\Cloud Share\AKS HCI`, the cloud agent cluster service will fail to start with the following (or similar) error message:

```powershell
Failed to start the cloud agent generic cluster service in failover cluster. The cluster resource group os in the 'failed' state. Resources in 'failed' or 'pending' states: 'MOC Cloud Agent Service'
```
To work around this issue, use a path that does not include spaces, for example, `C:\CloudShare\AKS-HCI`.

## Set-AksHciConfig fails with WinRM errors, but shows WinRM is configured correctly
When running [Set-AksHciConfig](./reference/ps/./set-akshciconfig.md), you might encounter the following error:

```powershell
WinRM service is already running on this machine.
WinRM is already set up for remote management on this computer.
Powershell remoting to TK5-3WP08R0733 was not successful.
At C:\Program Files\WindowsPowerShell\Modules\Moc\0.2.23\Moc.psm1:2957 char:17
+ ...             throw "Powershell remoting to "+$env:computername+" was n ...
+                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OperationStopped: (Powershell remo...not successful.:String) [], RuntimeException
    + FullyQualifiedErrorId : Powershell remoting to TK5-3WP08R0733 was not successful.
```

This error usually occurs as a result of a change in the user's security token (due to a change in group membership), a password change, or an expired password. In most cases, the issue can be remediated by logging off from the computer and logging back in. If this still fails, you can file an issue at [GitHub AKS HCI issues](https://aka.ms/aks-hci/issues).

## Install-AksHci failed on a multi-node installation with the error `Nodes have not reached active state`

When running [Install-AksHci](./reference/ps/install-akshci.md) on a single-node setup, the installation worked, but when setting up the failover cluster, the installation fails with the error message. However, pinging the cloud agent showed the CloudAgent was reachable.

To ensure all nodes can resolve the CloudAgent's DNS, run the following command on each node:

```powershell
Resolve-DnsName <FQDN of cloudagent>
```

When the step above succeeds on the nodes, make sure the nodes can reach the CloudAgent port to verify that a proxy is not trying to block this connection and the port is open. To do this, run the following command on each node:

```powershell
Test-NetConnection  <FQDN of cloudagent> -Port <Cloudagent port - default 65000>
```

## Install-AksHci timed out with the error `Waiting for API server`

After running [Install-AksHci](./reference/ps/install-akshci.md), the installation stopped and displayed the following error message:

```Output
\kubectl.exe --kubeconfig=C:\AksHci\0.9.7.3\kubeconfig-clustergroup-management 
get akshciclusters -o json returned a non zero exit code 1 
[Unable to connect to the server: dial tcp 192.168.0.150:6443: 
connectex: A connection attempt failed because the connected party 
did not properly respond after a period of time, or established connection 
failed because connected host has failed to respond.]
```

There are multiple reasons why an installation might fail with the `waiting for API server` error. See the following sections for possible causes and solutions for this error.

### Reason 1: Incorrect IP gateway configuration
If you're using static IP and you received the following error message, confirm that the configuration for the IP address and gateway is correct. 
```PowerShell
Install-AksHci 
C:\AksHci\kvactl.exe create --configfile C:\AksHci\yaml\appliance.yaml  --outfile C:\AksHci\kubeconfig-clustergroup-management returned a non zero exit code 1 [ ]
```

To check whether you have the right configuration for your IP address and gateway, run the following: 

```powershell
ipconfig /all
```

In the displayed configuration settings, confirm the configuration. You could also attempt to ping the IP gateway and DNS server. 

If these methods don't work, use [New-AksHciNetworkSetting](./reference/ps/new-akshcinetworksetting.md) to change the configuration.

### Reason 2: Incorrect DNS server
If you're using static IP, confirm that the DNS server is correctly configured. To check the host's DNS server address, use the following command:

```powershell
Get-NetIPConfiguration.DNSServer | ?{ $_.AddressFamily -ne 23} ).ServerAddresses
```

Confirm that the DNS server address is the same as the address used when running `New-AksHciNetworkSetting` by running the following command:

```powershell
Get-MocConfig
```

If the DNS server has been incorrectly configured, reinstall AKS on Azure Stack HCI with the correct DNS server. For more information, see [Restart, remove, or reinstall Azure Kubernetes Service on Azure Stack HCI ](./restart-cluster.md).

The issue was resolved after deleting the configuration and restarting the VM with a new configuration.

## Install-AksHci failed with the error `Failed to wait for addon arc-onboarding`

This error message appears after running [Install-AksHci](./reference/ps/install-akshci.md). To resolve this issue, use the following steps:

1. Open PowerShell and run [Uninstall-AksHci](./reference/ps/uninstall-akshci.md).
2. Open the Azure portal and navigate to the resource group you used when running `Install-AksHci`.
3. Check for any connected cluster resources that appear in a _Disconnected_ state and include a name shown as a randomly generated GUID. 
4. Delete these cluster resources.
5. Close the PowerShell session and open new session before running `Install-AksHci` again.

## After a failed installation, running Install-AksHci does not work
If your installation fails using [Install-AksHci](./reference/ps/uninstall-akshci.md), you should run [Uninstall-AksHci](./reference/ps/uninstall-akshci.md) before running `Install-AksHci` again. This issue happens because a failed installation may result in leaked resources that have to be cleaned up before you can install again.

## The AKS on Azure Stack HCI download package fails with the error: `msft.sme.aks couldn't load`
If you get this error, and the error message also indicates that loading chunks failed, you should use the latest version of Microsoft Edge or Google Chrome and try again.

## Error: `Waiting for pod 'Cloud Operator' to be ready`
When attempting to deploy an AKS on Azure Stack HCI cluster on an Azure VM, the installation was stuck at `Waiting for pod 'Cloud Operator' to be ready...`, and then failed and timed out after two hours. Attempts to troubleshoot by checking the gateway and DNS server showed they were working appropriately. Checks to see if there was an IP or MAC address conflict showed none were found. When viewing the logs, it showed that the VIP pool had not reached the logs. There was a restriction on pulling the container image using `sudo docker pull ecpacr.azurecr.io/kube-vip:0.3.4` that returned a Transport Layer Security (TLS) timeout instead of _unauthorized_. 

To resolve this issue, run the following steps:

1. Start to deploy your cluster.
2. When deployed, connect to management cluster VM through SSH as shown below:

   ```
   ssh -i (Get-MocConfig)['sshPrivateKey'] clouduser@<IP Address>
   ```

3. Change the maximum transmission unit (MTU) setting. Don't hesitate to make the change because if you make the change too late, then the deployment fails. Modifying the MTU setting helps unblock the container image pull.

   ```
   sudo ifconfig eth0 mtu 1300
   ```

4. To view the status of your containers, run the following command:
   ```
   sudo docker ps -a
   ```

After performing these steps, the container image pull should be unblocked.

## When running Set-AksHciRegistration, an error `Unable to check registered Resource Providers` appears

This error appears after running [Set-AksHciRegistration](./reference/ps/set-akshciregistration.md) in an AKS on Azure Stack HCI installation. The error indicates the Kubernetes Resource Providers are not registered for the tenant that is currently logged in.

To resolve this issue, run either the Azure CLI or the PowerShell steps below:

```azurecli
az provider register --namespace Microsoft.Kubernetes
az provider register --namespace Microsoft.KubernetesConfiguration
```

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.Kubernetes
Register-AzResourceProvider -ProviderNamespace Microsoft.KubernetesConfiguration
```

The registration takes approximately 10 minutes to complete. To monitor the registration process, use the following commands.

```azurecli
az provider show -n Microsoft.Kubernetes -o table
az provider show -n Microsoft.KubernetesConfiguration -o table
```

```powershell
Get-AzResourceProvider -ProviderNamespace Microsoft.Kubernetes
Get-AzResourceProvider -ProviderNamespace Microsoft.KubernetesConfiguration
```

## Error: `Install-Moc failed. Logs are available C:\Users\xxx\AppData\Local\Temp\v0eoltcc.a10`

If you receive this error when running [Install-AksHci](./reference/ps/install-akshci.md), you can get more information by running `$error[0].Exception.InnerException`. 

## When deploying AKS on Azure Stack HCI with a misconfigured network, deployment timed out at various points
When deploying AKS on Azure Stack HCI, the deployment may time out at different points of the process depending on where the misconfiguration occurred. You should review the error message to determine the cause and where it occurred.

For example, in the following error, the point at which the misconfiguration occurred is in `Get-DownloadSdkRelease -Name "mocstack-stable"`: 

```
$vnet = New-AksHciNetworkSettingSet-AksHciConfig -vnet $vnetInstall-AksHciVERBOSE: 
Initializing environmentVERBOSE: [AksHci] Importing ConfigurationVERBOSE: 
[AksHci] Importing Configuration Completedpowershell : 
GetRelease - error returned by API call: 
Post "https://msk8s.api.cdp.microsoft.com/api/v1.1/contents/default/namespaces/default/names/mocstack-stable/versions/0.9.7.0/files?action=generateDownloadInfo&ForegroundPriority=True": 
dial tcp 52.184.220.11:443: connectex: 
A connection attempt failed because the connected party did not properly
respond after a period of time, or established connection failed because
connected host has failed to respond.At line:1 char:1+ powershell -command
{ Get-DownloadSdkRelease -Name "mocstack-stable"}
```

This indicates that the physical Azure Stack HCI node can resolve the name of the download URL, `msk8s.api.cdp.microsoft.com`, but the node can't connect to the target server.

To resolve this issue, you need to determine where the breakdown occurred in the connection flow. Here are some steps to try to resolve the issue from the physical cluster node:

1. Ping the destination DNS name: ping `msk8s.api.cdp.microsoft.com`. 
2. If you get a response back and no time-out, then the basic network path is working. 
3. If the connection times out, then there could be a break in the data path. For more information, see [check proxy settings](./set-proxy-settings.md). Or, there could be a break in the return path, so you should check the firewall rules. 

## PowerShell deployment doesn't check for available memory before creating a new workload cluster

The **Aks-Hci** PowerShell commands do not validate the available memory on the host server before creating Kubernetes nodes. This issue can lead to memory exhaustion and virtual machines that do not start. This failure is currently not handled gracefully, and the deployment will stop responding with no clear error message. If you have a deployment that stops responding, open Event Viewer and check for a Hyper-V-related error message indicating there's not enough memory to start the VM.

## Error: unable to reconcile virtual network or Error: Install-Moc failed with error - Exception [[Moc] This machine does not appear to be configured for deployment]

You can trigger these errors when you run `Install-AksHci` without running [Set-AksHciConfig](./reference/ps/set-akshciconfig.md). To resolve the error, run `uninstall-akshci` and close all PowerShell windows. Open a new PowerShell session, and restart your AKS-HCI installation process by following [installing AKS-HCI using PowerShell](/azure-stack/aks-hci/kubernetes-walkthrough-powershell).

## Next steps

- [Troubleshooting overview](troubleshoot-overview.md)
- [Windows Admin Center known issues](known-issues-windows-admin-center.md)
- [Troubleshooting Kubernetes clusters](https://kubernetes.io/docs/tasks/debug-application-cluster/troubleshooting/)

If you continue to run into problems when you're using Azure Kubernetes Service on Azure Stack HCI, you can file bugs through [GitHub](https://aka.ms/aks-hci-issues).
