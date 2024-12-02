# Riverbed Operator


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

# Install cert-manager
The Riverbed Operator requires that  [cert-manager](https://cert-manager.io/docs/installation/) is installed in your cluster and that your cluster uses Kubernetes version 1.26 or greater. The cert-manager is used for auto-instrumentation of java and .Net applications.

**Check if cert-manager is installed on your cluster**
```
kubectl get pods --namespace cert-manager
```
**Install  [cert-manager](https://cert-manager.io/docs/installation/) in your cluster**
```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.5/cert-manager.yaml
```
Before installing the Riverbed Operator make sure the cert-manager-webhook is fully installed.
Some installs may take up to thirty seconds to complete. Run the command below and verify READY is 1/1
```
kubectl get pod -n cert-manager -l app.kubernetes.io/name=webhook
NAME                                    READY   STATUS    RESTARTS      AGE
cert-manager-webhook-5778696f85-4l7l4   1/1     Running   2 (30h ago)   5d5h
```
# Install the Riverbed Operator
Run the following from a command line.

```
kubectl apply -f https://github.com/riverbed/riverbed-operator/releases/latest/download/riverbed-operator.yaml
```

# Configure the Riverbed Operator


The Customer ID and Analysis Server Host will need to be configured for the APM Agent. Additional configuration may be required (outlined below)


```
kubectl create -f https://github.com/riverbed/riverbed-operator/latest/download/riverbed_configuration.yaml --namespace=riverbed-operator --edit
```

Under the ‘spec’ section of the file:
-
- update the analysisServerHost to your Analysis Server Host identifier.
- update the customerId to your Customer ID.   If using on-prem analysis server, leave this value as an empty string.

**Verify that the Riverbed APM Agent is running**

```
kubectl get pods --namespace riverbed-operator
```

A Riverbed APM Agent pod will be running for each node:

```
NAME                                                  READY   STATUS    RESTARTS   AGE
riverbed-apm-agent-2hpcs                               1/1     Running   0          6m36s
riverbed-apm-agent-54v54                               1/1     Running   0          6m36s
riverbed-operator-controller-manager-d44c57448-8jdth   2/2     Running   0          19m
```

# Auto-Instrument Your Applications

Update your application's `spec.template.metadata.annotations` to include one or more of the annotations listed in the below table:


| Annotation                            | Values                        | Defaults            | Description                                            |
|---------------------------------------|-------------------------------|---------------------|--------------------------------------------------------|
| instrument.apm.riverbed/inject-java   | "true" or "false"             | "false"             | For Java instrumentation                               |
| instrument.apm.riverbed/inject-dotnet | "true" or "false"             | "false"             | For .Net instrumentation                               |
| instrument.apm.riverbed/configName    | "Configuration Name"          | Operator configName | Process Configuration Name to instrument application.  |
| instrument.apm.riverbed/runtime       | "linux-musl-x64" or "linux-x64" | "linux-x64"       | Runtime environment used to instrument the application.  If your app is based on Alpine Linux you need to add the annotation to use linux-musl-x64 runtime instead of the default.|

## Example instrumented java application deployment:
This shows adding the annotation into a deployment file.

```
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      annotations:
        instrument.apm.riverbed/inject-java: "true"
      labels:
        app: spring-app
    spec:
      containers:
      - name: spring-app
        image: springio/gs-spring-boot-docker
        ports:
        - containerPort: 8080
EOF
```
If your application is already running or you do not want to add annotation to your deployment file. You can patch an existing deployment.

## Example instrumented dotnet application patch:
Here we are patching an existing dotnet deployment
```
kubectl patch deployment <application-deployment-name> -p '{"spec": {"template":{"metadata":{"annotations":{"instrument.apm.riverbed/inject-dotnet":"true"}}}} }'
```
**For Windows**
```
kubectl patch deployment <application-deployment-name> -p '{\"spec\": {\"template\":{\"metadata\":{\"annotations\":{\"instrument.apm.riverbed/inject-dotnet\":\"true\"}}}} }'
```
## Example instrumented java application patch:
Here we are patching an existing java deployment
```
kubectl patch deployment <application-deployment-name> -p '{"spec": {"template":{"metadata":{"annotations":{"instrument.apm.riverbed/inject-java":"true"}}}} }'
```
**For Windows**
```
kubectl patch deployment <application-deployment-name> -p '{\"spec\": {\"template\":{\"metadata\":{\"annotations\":{\"instrument.apm.riverbed/inject-java\":\"true\"}}}} }'
```
## Example instrumented alpine application patch:
If you are patching an existing alpine deployment, this additional patch is necessary, after you've applied the dotnet or java patches above:
```
kubectl patch deployment <application-deployment-name> -p '{"spec": {"template":{"metadata":{"annotations":{"instrument.apm.riverbed/runtime":"linux-musl-x64"}}}} }'
```
**For Windows**
```
kubectl patch deployment <application-deployment-name> -p '{\"spec\": {\"template\":{\"metadata\":{\"annotations\":{\"instrument.apm.riverbed/runtime\":\"linux-musl-x64\"}}}} }'
```


# Legal

© 2024 Riverbed Technology LLC All rights reserved.

Riverbed and any Riverbed product or service name or logo used herein are trademarks of Riverbed. All other trademarks used herein belong to their respective owners. The trademarks and logos displayed herein cannot be used without the prior written consent of Riverbed or their respective owners.

This product (“Product”) and this document together with the end user or technical documentation pertaining to the Product that is provided by Riverbed with the Product or otherwise made available by Riverbed (collectively, the “Documentation”) are licensed subject to the terms and conditions of the Riverbed customer agreement available at https://www.riverbed.com/license, including without limitation the additional terms and conditions set forth at https://www.riverbed.com/license/additional_use_rights.   No other rights or licenses are granted to the Product or Documentation except as set forth in the foregoing terms and conditions.  Riverbed and its licensors and suppliers retain all right, title, and interest in and to the Product and Documentation.

To the extent that this Product contains any third party or open source software, any required notices or licenses applicable to such third party or open source software will be included in the Documentation available on the Riverbed Support site at https://support.riverbed.com or within the Product itself.
