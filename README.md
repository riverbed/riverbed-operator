# Riverbed Operator

# Getting started
Welcome to the Riverbed Operator installation guide. This guide will quickly show you how to install the Riverbed Operator and instrument both Java and .NET applications running in Kubernetes.

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

**For OpenShift clusters - Attach to your OpenShift cluster**
```
oc login --token <token> --server <cluster-uri>
```

# Support for Red Hat OpenShift
The Riverbed Operator 2.0.0+ brings support for Red Hat OpenShift 4.13+ (Kubernetes 1.26+).

`oc` is a superset of `kubectl`. `kubectl` commands work in OpenShift if you replace `kubectl` with `oc`.

For simplicity, `kubectl` is used in the instructions below and will work in both OpenShift and non-OpenShift environments.

# Install cert-manager
The Riverbed Operator requires that  [cert-manager](https://cert-manager.io/docs/installation/) is installed in your cluster and that your cluster uses Kubernetes version 1.26 or greater. The cert-manager is used for auto-instrumentation of Java and .NET applications.

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

# Installing the Riverbed Operator
Run the following from a command line to install the Riverbed Operator.
```
kubectl apply -f https://raw.githubusercontent.com/riverbed/riverbed-operator/v2_alpha/riverbed-operator.yaml
```

# Upgrading the Riverbed Operator
When upgrading from a release prior to 2.0.0 you must uninstall the operator first by running the following from a command line before following the install instructions above.
```
kubectl delete -f https://raw.githubusercontent.com/riverbed/riverbed-operator/v2_alpha/riverbed-operator.yaml
```
Minor 2.x upgrades can be applied by simply running the install instructions above.

After an upgrade, you should restart any instrumented applications that were started prior to the upgrade.

# Configure the Riverbed Operator
The Customer ID and Analysis Server Host will need to be configured for the APM Agent. Additional configuration may be required (outlined below)
```
kubectl create -f https://raw.githubusercontent.com/riverbed/riverbed-operator/v2_alpha/riverbed_configuration.yaml --namespace=riverbed-operator --edit
```

Under the ‘spec’ section of the file:
- update the analysisServerHost to your Analysis Server Host identifier.
- update the customerId to your Customer ID.   If using on-prem analysis server, leave this value as an empty string.
- update proxyServerHost .  If you need to connect to the analysis server through a proxy, specify the fully-qualified domain name (FQDN), or IP address for the proxyServerHost.  Leave this value as an empty string if a proxy server is not used.
- update proxyServerPort if you are using a proxy server to the appropriate value, 

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
| instrument.apm.riverbed/inject-dotnet | "true" or "false"             | "false"             | For .NET instrumentation                               |
| instrument.apm.riverbed/configName    | "Configuration Name"          | Operator configName | Process Configuration Name to instrument application.  |
| instrument.apm.riverbed/runtime       | "linux-musl-x64" or "linux-x64" | "linux-x64"       | Runtime environment used to instrument the application.  If your app is based on Alpine Linux you need to add the annotation to use linux-musl-x64 runtime instead of the default.|

## Example instrumented Java application deployment:
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

## Example instrumented .NET application patch:
Here we are patching an existing .NET deployment
```
kubectl patch deployment <application-deployment-name> -p '{"spec": {"template":{"metadata":{"annotations":{"instrument.apm.riverbed/inject-dotnet":"true"}}}} }'
```

**For Windows**
```
kubectl patch deployment <application-deployment-name> -p '{\"spec\": {\"template\":{\"metadata\":{\"annotations\":{\"instrument.apm.riverbed/inject-dotnet\":\"true\"}}}} }'
```

## Example instrumented Java application patch:
Here we are patching an existing Java deployment
```
kubectl patch deployment <application-deployment-name> -p '{"spec": {"template":{"metadata":{"annotations":{"instrument.apm.riverbed/inject-java":"true"}}}} }'
```

**For Windows**
```
kubectl patch deployment <application-deployment-name> -p '{\"spec\": {\"template\":{\"metadata\":{\"annotations\":{\"instrument.apm.riverbed/inject-java\":\"true\"}}}} }'
```

## Example instrumented alpine application patch:
If you are patching an existing alpine deployment, this additional patch is necessary, after you've applied the .NET or Java patches above:
```
kubectl patch deployment <application-deployment-name> -p '{"spec": {"template":{"metadata":{"annotations":{"instrument.apm.riverbed/runtime":"linux-musl-x64"}}}} }'
```

**For Windows**
```
kubectl patch deployment <application-deployment-name> -p '{\"spec\": {\"template\":{\"metadata\":{\"annotations\":{\"instrument.apm.riverbed/runtime\":\"linux-musl-x64\"}}}} }'
```

## Example application patch to disable Java instrumentation:
Here we are patching an existing Java deployment to disable instrumentation
```
kubectl patch deployment <application-deployment-name> --type=json -p='[{"op": "remove", "path": "/spec/template/metadata/annotations/instrument.apm.riverbed~1inject-java"}]'
```

## Example application patch to disable .NET instrumentation:
Here we are patching an existing .NET deployment to disable instrumentation
```
kubectl patch deployment <application-deployment-name> --type=json -p='[{"op": "remove", "path": "/spec/template/metadata/annotations/instrument.apm.riverbed~1inject-dotnet"}]'
```

# Legal
© 2025 Riverbed Technology LLC All rights reserved.

Riverbed and any Riverbed product or service name or logo used herein are trademarks of Riverbed. All other trademarks used herein belong to their respective owners. The trademarks and logos displayed herein cannot be used without the prior written consent of Riverbed or their respective owners.

This product (“Product”) and this document together with the end user or technical documentation pertaining to the Product that is provided by Riverbed with the Product or otherwise made available by Riverbed (collectively, the “Documentation”) are licensed subject to the terms and conditions of the Riverbed customer agreement available at https://www.riverbed.com/license, including without limitation the additional terms and conditions set forth at https://www.riverbed.com/license/additional_use_rights.   No other rights or licenses are granted to the Product or Documentation except as set forth in the foregoing terms and conditions.  Riverbed and its licensors and suppliers retain all right, title, and interest in and to the Product and Documentation.

To the extent that this Product contains any third party or open source software, any required notices or licenses applicable to such third party or open source software will be included in the Documentation available on the Riverbed Support site at https://support.riverbed.com or within the Product itself.