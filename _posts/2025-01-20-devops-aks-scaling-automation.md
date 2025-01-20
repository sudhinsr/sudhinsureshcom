---
title: Azure DevOps Pipeline for AKS Scale Management
date: 2025-01-20
categories: [DevOps]
tags: [aks, devops, kubernetes]
comments: true
---

## What is this about?

This document explains how to use an **Azure DevOps pipeline** to manage the replica count of an **AKS (Azure Kubernetes Service)** deployment dynamically during non-working hours and weekends. By scaling down replicas when the application is not in use, you can optimize resource usage and reduce unnecessary costs. During working hours, the pipeline scales up replicas to ensure availability.

## How?

We have created a **CD YAML** file that manages scaling operations:  
**[aks-scale.yaml](https://dev.azure.com/euromonitor/Passport/_git/DevOps?path=/pipelines/scheduled/aks-scale.yaml)**.  
This YAML file is utilized by two pipelines with different parameters:

### 1. **Scale Up**

- **Pipeline Name**: *AKS Scale Up Deployment Replica (0 to 1)*
- **Schedule**: 8:00 AM IST
- **Purpose**: Increases the replica count from **0 to 1** to make the application ready for use during working hours.

### 2. **Scale Down**

- **Pipeline Name**: *AKS Scale Down Deployment Replica (1 to 0)*
- **Schedule**: 9:00 PM IST
- **Purpose**: Reduces the replica count from **1 to 0** to minimize resource usage and reduce costs during non-working hours.

## YAML File Overview

### Trigger

- The pipeline trigger is configured directly within the Azure DevOps pipeline to ensure the YAML file is reusable.

### Parameters

- **deployments**: A list of AKS deployments to be scaled. This parameter allows flexibility to target specific deployments.

### Variables

- **aksConnection**: The service connection for the AKS cluster.
- **aksNamespace**: The namespace in the AKS cluster where the deployments reside.

### Example YAML File

```yaml
trigger: none

parameters:
  - name: deployments
    type: object
    default:
      - deployment1
      - deployment2

variables:
  aksConnection: 'Dev WE AKS (Passport)-Passport'
  aksNamespace: 'passport'

pool:
  name: 'Scale Set Ubuntu 20.04'

steps:
  - ${{ each deployment in parameters.deployments }}:
      - task: Kubernetes@1
        displayName: '${{ deployment }} Scale'
        continueOnError: true
        inputs:
          connectionType: 'Kubernetes Service Connection'
          kubernetesServiceEndpoint: ${{ variables.aksConnection }}
          namespace: ${{ variables.aksNamespace }}
          command: 'scale'
          arguments: 'deployment ${{ deployment }} --current-replicas=$(currentReplicaCount) --replicas=$(replicaCount)'
```

## Best Practices

1. **Monitor Resource Usage**: Regularly monitor the AKS cluster to ensure the scaling operations are effective and resource utilization aligns with expectations.
2. **Review Scheduling**: Ensure the scheduled times for scaling up and down align with your team's working hours and application usage patterns.
3. **Add Error Handling**: Incorporate additional error-handling mechanisms in the pipeline to gracefully manage failures during scaling operations.

## Next Steps

- Update the **connection** and **pool** sections with the correct configuration for your environment.
- Verify and configure the trigger in the YAML file or Azure DevOps pipeline UI to ensure it meets your requirements.
- Expand the deployment list in the YAML file to include all necessary AKS deployments.

