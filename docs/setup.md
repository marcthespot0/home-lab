# notes 
Cilium Install:
```sh
# INSTALL CILIUM
export cilium_applicationyaml=$(curl -sL "https://raw.githubusercontent.com/acelinkio/argocd-homelab/main/manifest/kube-system.yaml" | yq eval-all '. | select(.metadata.name == "cilium" and .kind == "Application")' -)
export cilium_name=$(echo "$cilium_applicationyaml" | yq eval '.metadata.name' -)
export cilium_chart=$(echo "$cilium_applicationyaml" | yq eval '.spec.source.chart' -)
export cilium_repo=$(echo "$cilium_applicationyaml" | yq eval '.spec.source.repoURL' -)
export cilium_namespace=$(echo "$cilium_applicationyaml" | yq eval '.spec.destination.namespace' -)
export cilium_version=$(echo "$cilium_applicationyaml" | yq eval '.spec.source.targetRevision' -)
export cilium_values=$(echo "$cilium_applicationyaml" | yq eval '.spec.source.helm.valuesObject' - | yq eval 'del(.gatewayAPI)' - | yq eval 'del(.ingressController)' -)
```

Create Cilium Values file to edit ipv4NativeRoutingCIDR 10.42.0.0./16 and k8sServiceHost: which will be the ip address of your endpoint.

```sh
echo "$cilium_values" > localvalues.yaml
#modify localvalues
helm template $cilium_name $cilium_chart --repo $cilium_repo --version $cilium_version --namespace $cilium_namespace -f localvalues.yaml | kubectl apply --filename -
```

Fresh install with Talos, I found I had to approve the csr's for each of my talos nodes before I was able to succesfully deploy ArgoCD

```sh
kubectl get csr
#Approve all pending csr's
kubectl certificate approve csr-8j8gw
```


```sh
kubectl create namespace argocd
kubectl create secret generic stringreplacesecret --namespace argocd --from-literal domain=$domain --from-literal cloudflaretunnelid=$cloudflaretunnelid --from-literal ciliumipamcidr=$ciliumipamcidr

kubectl create namespace 1passwordconnect
kubectl create secret generic 1passwordconnect --namespace 1passwordconnect --from-literal 1password-credentials.json="$onepasswordconnect_json"

kubectl create namespace external-secrets
kubectl create secret generic 1passwordconnect --namespace external-secrets --from-literal token=$externalsecrets_token```

```sh 
export argocd_applicationyaml=$(curl -sL "https://raw.githubusercontent.com/marcthespot0/home-lab/main/manifest/argocd.yaml" | yq eval-all '. | select(.metadata.name == "argocd" and .kind == "Application")' -)
export argocd_name=$(echo "$argocd_applicationyaml" | yq eval '.metadata.name' -)
export argocd_chart=$(echo "$argocd_applicationyaml" | yq eval '.spec.source.chart' -)
export argocd_repo=$(echo "$argocd_applicationyaml" | yq eval '.spec.source.repoURL' -)
export argocd_namespace=$(echo "$argocd_applicationyaml" | yq eval '.spec.destination.namespace' -)
export argocd_version=$(echo "$argocd_applicationyaml" | yq eval '.spec.source.targetRevision' -)
export argocd_values=$(echo "$argocd_applicationyaml" | yq eval '.spec.source.helm.valuesObject' - | yq eval 'del(.configs.cm)' -)
export argocd_config=$(curl -sL "https://raw.githubusercontent.com/marcthespot0/home-lab/main/manifest/argocd.yaml" | yq eval-all '. | select(.kind == "AppProject" or .kind == "ApplicationSet")' -)

# install
echo "$argocd_values" | helm template $argocd_name $argocd_chart --repo $argocd_repo --version $argocd_version --namespace $argocd_namespace --values - | kubectl apply --namespace $argocd_namespace --filename -

# configure
echo "$argocd_config" | kubectl apply --filename -
```


