# Open Telemetry K8s workshop - Bank of Anthos

## Step 1 - Prerequisites
- Hypervisor installation (VMWare, Hyper-V, Qemu,...)
- Ubuntu20.04 64bit (desktop or server) - 40gb disk, 12GB RAM
- Installation [Docker, Kind, Kubectl](https://github.com/dt-wv/k8s/tree/main/workshop/README.md)
- Access internet  
- Dynatrace tenant with Admin rights (token creation,...)

## Step 2 - Bank-of-Anthos application deployment
`$ sudo su -`  
`# apt-get install -y git`  
`# cd $HOME; git clone https://github.com/dt-wv/bank-of-anthos.git`  
`# cd bank-of-anthos`  
`# kubectl create ns bank-of-anthos-otel`  
`# kubectl apply -f ./extras/jwt/ --namespace=bank-of-anthos-otel`  
`# kubectl apply -f ./kubernetes-manifests/ --namespace=bank-of-anthos-otel`  
`# sleep 120 && kubectl get pods -n bank-of-anthos-otel`  

## Step 3 - install [Cert manager](https://cert-manager.io/docs/installation/kubectl/)
`# kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml`  
`# sleep 120 && kubectl get pods -n cert-manager`  
note: please wait 2min until the cert-manager finishes installation

### Optional step ([verify cert-manager installation](https://cert-manager.io/docs/installation/verify/ ))
  

## Step 4 - Opentelemetry [operator](https://github.com/open-telemetry/opentelemetry-operator) installation
`# kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/latest/download/opentelemetry-operator.yaml`  

## Step 5 - Opentelemetry Collector installation
`# curl -LO https://raw.githubusercontent.com/dt-wv/otel/main/collector/otel-dt-collector-deployment.yml`  
`# vi otel-dt-collector-deployment.yml` (add environment-id and API-Token values. API-Token scopes: ingest logs, metrics, OpenTelemetry traces)  
`# kubectl create ns otel-backend`  
`# kubectl apply -f otel-dt-collector-deployment.yml`  

## Step 6 - Install the Custom Resource Definition (CRD) for instrumentation
`# curl -LO https://raw.githubusercontent.com/dt-wv/otel/main/instrumentation/instrumentation.yml`  
`# sed -i 's/my-application-namespace/bank-of-anthos-otel/g' instrumentation.yml`  
`# kubectl apply -f instrumentation.yml`  

## Step 7 - Patch the Bank-of-Anthos spec for auto-instrumentation    
(patching is required to add the auto-instrumentation annotations to the pod specs where the technology supports it)  
`# kubectl patch deployment balancereader -n bank-of-anthos-otel -p '{"spec": {"template":{"metadata":{"annotations":{"instrumentation.opentelemetry.io/inject-java":"true"}}}} }'`  
`# kubectl patch deployment contacts -n bank-of-anthos-otel -p '{"spec": {"template":{"metadata":{"annotations":{"instrumentation.opentelemetry.io/inject-python":"true"}}}} }'`  
`# kubectl patch deployment frontend -n bank-of-anthos-otel -p '{"spec": {"template":{"metadata":{"annotations":{"instrumentation.opentelemetry.io/inject-python":"true"}}}} }'`  
`# kubectl patch deployment ledgerwriter -n bank-of-anthos-otel -p '{"spec": {"template":{"metadata":{"annotations":{"instrumentation.opentelemetry.io/inject-java":"true"}}}} }'`  
`# kubectl patch deployment transactionhistory -n bank-of-anthos-otel -p '{"spec": {"template":{"metadata":{"annotations":{"instrumentation.opentelemetry.io/inject-java":"true"}}}} }'`  
`# kubectl patch deployment userservice -n bank-of-anthos-otel -p '{"spec": {"template":{"metadata":{"annotations":{"instrumentation.opentelemetry.io/inject-python":"true"}}}} }'`  
`# for i in $(kubectl get deployments -n bank-of-anthos-otel | awk '{print $1}');do kubectl rollout restart deployment -n bank-of-anthos-otel $i;done`  
### verify patch has been applied
`# kubectl describe -n bank-of-anthos-otel deployment`  

## Step 8 - verify in Dynatrace - Distributed traces -> ingested traces
### Troubleshooting
Get the name of the otel-collector pod:  
`# kubectl get pod -n otel-backend`  
Logs of the otel collector (select the pod name from the previous command output):  
`# kubectl logs -n otel-backend otel-collector-collector-....`  
   

