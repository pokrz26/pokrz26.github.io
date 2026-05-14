---
title: Useful Kubernetes Commands
date: 2026-04-23 14:04:00 +0200
categories: [Kubernetes]
tags: [kubernetes, aks, kubectl, keda]
description: A collection of useful Kubernetes commands for managing AKS clusters and resources.
---

## Document Information

- **Azure CLI Version:** 2.84.0
- **PowerShell Version:** 7.6.1
- **Client Version:** v1.31.2
- **Kustomize Version:** v5.4.2
- **Server Version:** v1.33.6

## Login to Azure AKS

Use Azure CLI to download AKS cluster credentials and configure kubelogin.

```powershell
az aks get-credentials --resource-group rg-name --name aks-name
kubelogin convert-kubeconfig -l azurecli
```

## Manage kubeconfig contexts

View and switch between Kubernetes contexts stored in your kubeconfig.

### Get current context

```powershell
kubectl config current-context
```

### Get all contexts

```powershell
kubectl config get-contexts
```

### Get kubeconfig

```powershell
kubectl config view
```

### Change context

```powershell
kubectl config use-context context-name
```

### Fully delete context from kubeconfig

```powershell
kubectl config delete-cluster cluster-name-from-config-view
kubectl config delete-context context-name-from-config-view
kubectl config unset users.user-name-from-config-view
```

## Get resources with selected types

```powershell
kubectl get CronJobs,Jobs,Pods -n namespace-name
```

## Manage jobs

### Create Job from CronJob

```powershell
kubectl create job --from=cronjob/name-of-cron-job name-of-cron-job-test -n namespace-name
```

### Get live logs from Pod

```powershell
kubectl get pods -n namespace-name
kubectl logs pod-name-from-previous-step -n namespace-name -f
```

### Delete Job

```powershell
kubectl delete job name-of-cron-job-test -n namespace-name
```

## Check KEDA scaler logs

```powershell
kubectl get pods -n kube-system
kubectl logs keda-operator-<pod-name-from-previous-step> -n kube-system -f 
```

kubernetes - certmanager

kubectl get certificate -n defectdojo-prod
kubectl get certificaterequest -n defectdojo-prod
kubectl get challenges -n defectdojo-prod -w