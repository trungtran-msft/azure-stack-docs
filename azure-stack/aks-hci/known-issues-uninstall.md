---
title: Troubleshoot issues when uninstalling AKS on Azure Stack HCI. 
description: Get help to troubleshoot issues and errors when uninstalling AKS on Azure Stack HCI.
author: mattbriggs
ms.topic: troubleshooting
ms.date: 11/05/2021
ms.author: mabrigg 
ms.lastreviewed: 1/14/2022
ms.reviewer: abha

---

# Fix known issues and errors when uninstalling AKS on Azure Stack HCI

Use this topic to help you troubleshoot and resolve issues when uninstalling AKS on Azure Stack HCI.

## Running Remove-AksHciCluster results in the error: `A workload cluster with the name 'my-workload-cluster' was not found`

If you encounter this error when running [Remove-AksHciCluster](./reference/ps/remove-akshcicluster.md), you should check to make sure you have used the correct information for removing the cluster.

## Running Remove-AksHciCluster results in the error: `Error: unable to delete group clustergroup-spdb:...`

When running [Remove-AksHciCluster](./reference/ps/remove-akshcicluster.md), the following error occurs because there may be a deadlock:

`Error: unable to delete group clustergroup-spdb: failed to delete group clustergroup-spdb: rpc error: code = DeadlineExceeded desc = context deadline exceeded`

To resolve this issue, restart CloudAgent. 

## Uninstall-AksHciAdAuth fails with the error `[Error from server (NotFound): secrets "keytab-akshci-scale-reliability" not found]`
If [Uninstall-AksHciAdAuth](./reference/ps/./uninstall-akshciadauth.md) displays this error, you should ignore it for now as this issue will be fixed.

## Uninstall-AksHCI is not cleaning up cluster resources (`ownergroup ca-<GUID>`)
Due to insufficient permissions in Active Directory, [Uninstall-AksHci](./reference/ps/uninstall-akshci.md) could not remove cluster resource objects in Active Directory, which can lead to failures in subsequent deployments. To fix this issue, ensure that the user performing the installation has Full Control permissions to create/modify/remove Active Directory objects in the Active Directory container that the server and service objects are created in.

## Error occurs when running Uninstall-AksHci and AKS on Azure Stack HCI is not installed
If you run [Uninstall-AksHci](./reference/ps/./uninstall-akshci.md) when AKS on Azure Stack HCI is not installed, you'll receive the error message: _Cannot bind argument to parameter 'Path' because it is null_. You can safely ignore the error message as there is no functional impact.

## When using kubectl to delete a node, the associated VM might not be deleted

You'll see this issue if you follow these steps:

1. Create a Kubernetes cluster.
2. Scale the cluster to more than two nodes.
3. Delete a node by running the following command: 
   ```powershell
   kubectl delete node <node-name>
   ```

4. Return a list of the nodes by running the following command:
   ```powershell
   kubectl get nodes
   ```
   The removed node isn't listed in the output.
5. Open a PowerShell with administrative privileges and run the following command:
   ```powershell
   get-vm
   ```
The removed node is still listed.

This failure causes the system not to recognize that the node is missing, and therefore, a new node will not spin up. 

## Running the Remove-ClusterNode command evicts the node from the failover cluster, but the node still exists
When running the [Remove-ClusterNode](/powershell/module/failoverclusters/remove-clusternode?view=windowsserver2019-ps&preserve-view=true) command, the node is evicted from the failover cluster, but if [Remove-AksHciNode](./reference/ps/remove-akshcinode.md) is not run afterwards, the node will still exist in CloudAgent.

Since the node was removed from the cluster, but not from CloudAgent, if you use the VHD to create a new node, a _File not found_ error appears. This issue occurs because the VHD is in shared storage, and the evicted node does not have access to it.

To resolve this issue, remove a physical node from the cluster and then follow the steps below:

1. Run `Remove-AksHciNode` to de-register the node from CloudAgent.
2. Perform routine maintenance, such as re-imaging the machine.
3. Add the node back to the cluster.
4. Run `Add-AksHciNode` to register the node with CloudAgent.

## Next steps
- [Troubleshooting overview](troubleshoot-overview.md)
- [Installation issues and errors](known-issues-installation.md)
