# Useful Kubernetes commands

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
kubectl get CronJobs,Jobs,Pods -n repo-scanner
```

## Manage jobs

### Create Job from CronJob

```powershell
kubectl create job --from=cronjob/repo-scanner-reports-processor-cron-job repo-scanner-reports-processor-cron-job-test -n repo-scanner
```

### Get live logs from Pod

```powershell
kubectl logs repo-scanner-reports-processor-cron-job-test-5sc5b -n repo-scanner -f
```

### Delete Job

```powershell
kubectl delete job repo-scanner-reports-processor-cron-job-test -n repo-scanner
```
