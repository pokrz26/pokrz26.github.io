---
title: Useful Kubernetes Commands
date: 2026-07-14 00:00:00 +0200
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
az aks get-credentials --resource-group <resource-group-name> --name <aks-cluster-name>
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
kubectl config use-context <context-name>
```

### Fully delete context from kubeconfig

```powershell
kubectl config delete-cluster <cluster-name-from-config-view>
kubectl config delete-context <context-name-from-config-view>
kubectl config unset users.<user-name-from-config-view>
```

### Remove cached kubelogin tokens

Use this when you need to clear cached Azure authentication tokens and force a fresh login flow.

```powershell
kubelogin remove-tokens
```

## Get resources with selected types

```powershell
kubectl get CronJobs,Jobs,Pods -n <namespace-name>
```

## Manage jobs

### Create Job from CronJob

```powershell
kubectl create job --from=cronjob/<cron-job-name> <job-name> -n <namespace-name>
```

### Get live logs from Pod

```powershell
kubectl get pods -n <namespace-name>
kubectl logs <pod-name-from-previous-step> -n <namespace-name> -f
```

### Delete Job

```powershell
kubectl delete job <job-name> -n <namespace-name>
```

### Delete Jobs matching a name pattern

Use this to remove multiple jobs in the same namespace when they share a common prefix.

```powershell
kubectl get jobs -n <namespace-name> -o name | Where-Object { $_ -match "<job-name-prefix>" } | ForEach-Object { kubectl delete $_ -n <namespace-name> }
```

### Force delete a stuck Pod

Use this only when a pod is stuck in `Terminating` and a normal delete does not complete.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> `--force --grace-period=0` skips graceful shutdown. The container process may not finish cleanup, in-flight work can be lost, and attached systems may observe abrupt termination.
{: .prompt-warning }
<!-- markdownlint-restore -->

```powershell
kubectl delete pod <pod-name> -n <namespace-name> --force --grace-period=0
```

### Force delete Pods matching a name pattern

Use this to remove multiple stuck pods in the same namespace when they share a common prefix.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Apply the same caution as for single-pod force deletion. Prefer a normal `kubectl delete pod` first and use force deletion only for pods that do not terminate cleanly.
{: .prompt-warning }
<!-- markdownlint-restore -->

```powershell
kubectl get pods -n <namespace-name> -o name | Where-Object { $_ -match "<pod-name-prefix>" } | ForEach-Object { kubectl delete $_ -n <namespace-name> --force --grace-period=0 }
```

### Inspect processes running inside a Pod

Use this to inspect the process tree, parent process relationships, and wait channels inside a running container.

```powershell
kubectl exec -n <namespace-name> <pod-name> -- ps -eo pid,ppid,pgid,stat,wchan:32,args
```

## Check KEDA scaler logs

```powershell
kubectl get pods -n <keda-namespace>
kubectl logs <keda-operator-pod-name> -n <keda-namespace> -f
```

## Check cert-manager resources

Use these commands to troubleshoot certificate request failures in cert-manager.

### Get certificate information

```powershell
kubectl get certificate -n <namespace-name>
```

### Get certificate request information

```powershell
kubectl get certificaterequest -n <namespace-name>
```

### Watch certificate challenges

```powershell
kubectl get challenges -n <namespace-name> -w
```
