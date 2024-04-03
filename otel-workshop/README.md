# Open Telemetry K8s workshop

## Step 1 - Prerequisites
- Hypervisor installation (VMWare, Hyper-V, Qemu,...)
- Ubuntu20.04 64bit (desktop or server) - 40gb disk, 12GB RAM
- Installation [Docker, Kind, Kubectl](https://github.com/dt-wv/k8s/tree/main/workshop/README.md)
- Access internet  
- Dynatrace tenant with Admin rights (token creation,...)

## Step 2 - Easytravel application deployment
`$ sudo su -`  
`# curl -LO https://raw.githubusercontent.com/dt-wv/easyTravel/main/k8s.yaml`   
`# kubectl apply -f k8s.yaml`   
`# sleep 120 && kubectl get pods -n easytravel`  

## Step 3 - install [Cert manager](https://cert-manager.io/docs/installation/kubectl/)
`# kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml`  
`# sleep 120 && kubectl get pods -n cert-manager`  
note: please wait 2min until the cert-manager finishes installation

### Optional step ([verify cert-manager installation](https://cert-manager.io/docs/installation/verify/ ))
  

## Step 4 - Opentelemetry [operator](https://github.com/open-telemetry/opentelemetry-operator) installation
`# kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/latest/download/opentelemetry-operator.yaml`  

## Step 5 - Opentelemetry Collector installation
`# curl –LO https://raw.githubusercontent.com/dt-wv/otel/main/collector/otel-collector-deployment.yml > otel-collector-deployment.yaml`  
`# vi otel-collector-deployment.yaml` (add environment-id and API-Token values)    
`# kubectl create ns otel-backend`  
`# kubectl apply -f otel-collector-deployment.yaml`  

## Step 6 - Install the Custom Resource Definition (CRD) for instrumentation
`# curl -LO https://raw.githubusercontent.com/dt-wv/otel/main/instrumentation/instrumentation.yml`  
`# sed -i 's/my-application-namespace/easytravel/g' instrumentation.yml`  
`# kubectl apply -f instrumentation.yml` 

## Step 7 - Patch the EasyTravel spec for auto-instrumentation    
(patching is required to add the auto-instrumentation annotations to the pod specs where the technology supports it)  
`# kubectl patch deployment easytravel-backend -n easytravel -p '{"spec": {"template":{"metadata":{"annotations":{"instrumentation.opentelemetry.io/inject-java":"true"}}}} }'
`  
`# kubectl patch deployment easytravel-frontend -n easytravel -p '{"spec": {"template":{"metadata":{"annotations":{"instrumentation.opentelemetry.io/inject-java":"true"}}}} }'
`  
`# kubectl patch deployment easytravel-angular-frontend -n easytravel -p '{"spec": {"template":{"metadata":{"annotations":{"instrumentation.opentelemetry.io/inject-java":"true"}}}} }'
`  
`# kubectl rollout restart deployment -n easytravel easytravel-backend`  
`# kubectl rollout restart deployment -n easytravel easytravel-frontend`  
`# kubectl rollout restart deployment -n easytravel easytravel-angular-frontend`    
### verify patch has been applied
`# kubectl describe -n easytravel deployment easytravel-backend `

## Step 8 - verify in Dynatrace - Distributed traces -> ingested traces
### Troubleshooting
Get the name of the otel-collector pod:  
`# kubectl get pod -n otel-backend`  
Logs of the otel collector (select the pod name from the previous command output):  
`# kubectl logs -n otel-backend otel-collector-collector-....`  
   

