# Dynatrace OneAgent + Otel K8s Workshop

# Dynatrace OneAgent

## Step 1 - Prerequisites
- Hypervisor installation (VMWare, Hyper-V, Qemu,...)
- Ubuntu20.04 64bit (desktop or server) - 40Gb disk, 12GB RAM
- Installation [Docker, Kind, Kubectl](https://github.com/dt-wv/k8s/tree/main/workshop/README.md)
- Access internet  
- Dynatrace tenant with Admin rights (token creation,...)

## Step 2 - Bank-of-Anthos application deployment
`$ sudo su -`  
`# apt-get install -y git`  
`# cd $HOME; git clone https://github.com/dt-wv/bank-of-anthos.git`  
`# cd bank-of-anthos`  
`# kubectl create ns bank-of-anthos`  
`# kubectl apply -f ./extra/jwt/ --namespace=bank-of-anthos`  
`# kubectl apply -f ./kubernetes-manifests/ --namespace=bank-of-anthos`  
`# sleep 120 && kubectl get pods -n bank-of-anthos`  

## Step 3 - install Dynatrace as ApplicationOnly
`$ sudo su -`

Go to the link of the Dynatrace documentation for a [ApplicationOnly](https://docs.dynatrace.com/docs/setup-and-configuration/setup-on-k8s/installation/app-observability-automated#manifest) installation and execute all steps until step 4.  
On Step 2 take <b>without</b> CSI driver.  

<b>Operator token scopes:</b>  
![](img/operator_k8s_token_scopes.jpg)  

<b>Data ingest token scopes:</b>  
![](img/dataingest_token_scopes.jpg)  

Dynakube can be downloaded and please replace the 'name' and 'apiurl' with the correct value.  
`# curl -LO https://raw.githubusercontent.com/dt-wv/k8s/main/ApplicationMonitoring/dynakube-applicationMonitoring-generic-without-csi.yml`  
`# vi dynakube-applicationMonitoring-generic-without-csi.yml`  
`# kubectl apply -f dynakube-applicationMonitoring-generic-without-csi.yml`

## Step 4 - Modify the namespace  
`# kubectl patch ns bank-of-anthos -p '{"metadata":{"labels":{"instrumentation":"oneagent"}}}'`

## Step 5 - Restart the deployments
`# for i in $(kubectl get deployments -n bank-of-anthos | awk '{print $1}'); do kubectl rollout restart deployment -n bank-of-anthos $1; done`

# OpenTelemetry

## Step 1 - install [Cert manager](https://cert-manager.io/docs/installation/kubectl/)
`# kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml`  
`# sleep 120 && kubectl get pods -n cert-manager`  
note: please wait 2min until the cert-manager finishes installation

### Optional step ([verify cert-manager installation](https://cert-manager.io/docs/installation/verify/ ))
  

## Step 2 - Opentelemetry [operator](https://github.com/open-telemetry/opentelemetry-operator) installation
`# kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/latest/download/opentelemetry-operator.yaml`  

## Step 3 - Opentelemetry Collector installation
`# curl -LO https://raw.githubusercontent.com/dt-wv/otel/main/collector/otel-dt-collector-deployment-simple.yml`  
`# vi otel-dt-collector-deployment-simple.yml` (add environment-id and API-Token values. API-Token scopes: ingest logs, metrics, OpenTelemetry traces)  
`# kubectl create ns otel-backend`  
`# kubectl apply -f otel-dt-collector-deployment-simple.yml`  

## Step 4 - Install the Custom Resource Definition (CRD) for instrumentation
`# curl -LO https://raw.githubusercontent.com/dt-wv/otel/main/instrumentation/instrumentation.yml`  
`# sed -i 's/my-application-namespace/bank-of-anthos/g' instrumentation.yml`  
`# kubectl apply -f instrumentation.yml`  

## Step 5 - Patch the Bank-of-Anthos spec for auto-instrumentation where the OneAgent doesnt support instrumentation   
(patching is required to add the auto-instrumentation annotations to the pod specs where the technology supports it)   
`# kubectl patch deployment contacts -n bank-of-anthos -p '{"spec": {"template":{"metadata":{"annotations":{"instrumentation.opentelemetry.io/inject-python":"true"}}}} }'`  
`# kubectl patch deployment frontend -n bank-of-anthos -p '{"spec": {"template":{"metadata":{"annotations":{"instrumentation.opentelemetry.io/inject-python":"true"}}}} }'`   
`# kubectl patch deployment userservice -n bank-of-anthos -p '{"spec": {"template":{"metadata":{"annotations":{"instrumentation.opentelemetry.io/inject-python":"true"}}}} }'`  
`# for i in $(kubectl get deployments -n bank-of-anthos | awk '{print $1}');do kubectl rollout restart deployment -n bank-of-anthos-otel $i;done`  
### verify patch has been applied
`# kubectl describe -n bank-of-anthos deployment`  

## Step 6 - verify in Dynatrace - Distributed traces -> ingested traces
### Troubleshooting
Get the name of the otel-collector pod:  
`# kubectl get pod -n otel-backend`  
Logs of the otel collector (select the pod name from the previous command output):  
`# kubectl logs -n otel-backend otel-collector-collector-....`  
