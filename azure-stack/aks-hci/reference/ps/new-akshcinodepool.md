---
title: New-AksHciNodePool for AKS on Azure Stack HCI
author: mattbriggs
description: The New-AksHciNodePool PowerShell command creates a new node pool to an existing cluster
ms.topic: reference
ms.date: 7/20/2021
ms.author: mabrigg 
ms.lastreviewed: 1/14/2022
ms.reviewer: jeguan

---

# New-AksHciNodePool

## Synopsis
Create a new node pool to an existing cluster.

## Syntax
```powershell
New-AksHciNodePool -clusterName <String>
                   -name <String>
                  [-count <int>]
                  [-osType <String>]
                  [-vmSize <VmSize>]
                  [-taints <Taint>]
                  [-maxPodCount <int>]
```

## Description
Create a new node pool to an existing cluster.

## Examples

### Create a new node pool with default parameters

```powershell
PS C:\> New-AksHciNodePool -clusterName mycluster -name nodepool1
```

### Create a Linux node pool

```powershell
PS C:\> New-AksHciNodePool -clusterName mycluster -name linuxnodepool -osType linux
```

### Create a Windows node pool

```powershell
PS C:\> New-AksHciNodePool -clusterName mycluster -name windowsnodepool -osType windows
```

### Create a node pool with custom VM size

```powershell
PS C:\> New-AksHciNodePool -clusterName mycluster -name nodepool1 -vmSize Standard_A2_v2
```

### Create a node pool with taints

```powershell
PS C:\> New-AksHciNodePool -clusterName mycluster -name nodepool1 -taints sku=gpu:NoSchedule
```

### Create a node pool with max pod count

```powershell
PS C:\> New-AksHciNodePool -clusterName mycluster -name nodepool1 -maxPodCount 100
```

## Parameters

### -clusterName
The name of the existing Kubernetes cluster.

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: True
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -name
The name of your node pool. The node pool name must not be the same as another existing node pool.

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: True
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -count
The node count of your node pool. Defaults to 1.

```yaml
Type: System.Int32
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -osType
The OS type of the nodes in your node pool. Defaults to Linux.

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: Linux
Accept pipeline input: False
Accept wildcard characters: False
```

### -vmSize
The VM size of the nodes in your node pool. Defaults to Standard_K8S3_v1. To get the available VM sizes, use the [Get-AksHciVmSize](get-akshcivmsize.md) command.

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: Standard_K8S3_v1
Accept pipeline input: False
Accept wildcard characters: False
```

### -taints
The node taints for the node pool. You can't change the node taints after the node pool is created.

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -maxPodCount
The maximum number of pods deployable to a node. This number needs to be greater than 50.

```yaml
Type: System.Int32
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: 110
Accept pipeline input: False
Accept wildcard characters: False
```

## Next steps

[AksHci PowerShell Reference](index.md)