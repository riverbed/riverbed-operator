# riverbed-operator
 

# Getting started
Welcome to the Riverbed Operator installation guide. This guide will quickly show you how to install the Riverbed Operator and instrument both Java and .Net applications running in Kubernetes.

# Attach to your cluster
Ensure that kubectl points to your Kubernetes cluster where the Riverbed Operator and Riverbed APM Agent will run.

**For Azure clusters - Attach to your Azure cluster**
```
az aks get-credentials --resource-group <your-resource-group> --name <your-aks-cluster-name>
```

**For AWS clusters - Attach to your AWS cluster**
```
aws eks --region <region-name> update-kubeconfig --name <cluster-name>
```

# Install a Certificate Manager
The Riverbed Operator requires that a certificate manager is installed in your cluster and that your cluster uses Kubernetes version 1.22 or greater. The Certificate Manager is used for auto-instrumentation of java and .Net applications. 

**Check if a certificate manager is installed on your cluster**
```
kubectl get pods --namespace cert-manager
```
**Install a certificate manager in your cluster**
```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.5/cert-manager.yaml
```
Before installing the Riverbed Operator make sure the Certificate Manager is fully installed. 
Some certificate manager installs may take up to thirty seconds to procecss. Run the command below and verify that you see three running processes.
```
kubectl get pods --namespace cert-manager 
NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-5bd57786d4-wjbvn              1/1     Running   0          6s
cert-manager-cainjector-57657d5754-vj6cb   1/1     Running   0          6s
cert-manager-webhook-7d9f8748d4-f8g58      1/1     Running   0          6s
```
# Install the Riverbed Operator
Run the following from a command line. 

```
kubectl apply -f https://raw.githubusercontent.com/rvbd-operator/beta/1.0.0/riverbed-operator.yaml
```

# Configure the Riverbed Operator


The Customer ID and Analysis Server Host will need to be configured for the APM Agent. Additional configuration may be required (outlined below)


```
kubectl create -f https://raw.githubusercontent.com/rvbd-operator/beta/1.0.0/riverbed_configuration_betav1.yaml --namespace=riverbed-operator --edit
```

Under the ‘spec’ section of the file:
- 
- update the analysisServerHost to your Analysis Server Host identifier.
- update the customerId to your Customer ID.   If using on-prem analysis server, leave this value as an empty string.
**Verify that the Riverbed APM Agent is running**

```
kubectl get pods -n riverbed-operator
```

A Riverbed APM Agent pod will be running for each node:

```
NAME                                                  READY   STATUS    RESTARTS   AGE
riverbed-apm-agent-2hpcs                               1/1     Running   0          6m36s
riverbed-apm-agent-54v54                               1/1     Running   0          6m36s
riverbed-operator-controller-manager-d44c57448-8jdth   2/2     Running   0          19m
```


# Configuring instrumentation for Java example
**Deploy tomcat**
```
kubectl apply -f https://raw.githubusercontent.com/rvbd-operator/beta/1.0.0/example/tomcat.yaml
```

