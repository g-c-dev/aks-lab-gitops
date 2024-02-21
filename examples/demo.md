# preparation 

## create tempo folders

3 tabs, cmd+shift+i names: platform-team, team-alice, team-bob


tab: team-alice
```shell
mkdir -p /tmp/alice && cd /tmp/alice
export AZURE_CONFIG_DIR=$(pwd)
az login 
```

other tab: team-bob
```shell
mkdir -p /tmp/bob && cd /tmp/bob
export AZURE_CONFIG_DIR=$(pwd)
az login 
```

## create simpla kubeconfig 

```shell
cat <<EOT > /tmp/alice/kubeconfig.yaml
apiVersion: v1
kind: Config
clusters:
  - cluster:
      server: https://capsule.k01.labs.g-c.dev
    name: capsule-api-server
contexts:
  - context:
      cluster: capsule-api-server
      user: tenant-user
    name: capsule-context
current-context: capsule-context
users:
  - name: tenant-user
    user:
      exec:
        apiVersion: client.authentication.k8s.io/v1beta1
        args:
          - get-token
          - --login
          - azurecli
          - --server-id
          - 6dae42f8-4368-4678-94ff-3960e28e3630 # <-- well-known server-id for Azure AKS service
        command: kubelogin
EOT

export KUBECONFIG=/tmp/alice/kubeconfig.yaml
k config set-context --current --namespace=app-alice-default
```

now for team-bob

```shell
cat <<EOT > /tmp/bob/kubeconfig.yaml
apiVersion: v1
kind: Config
clusters:
  - cluster:
      server: https://capsule.k01.labs.g-c.dev
    name: capsule-api-server
contexts:
  - context:
      cluster: capsule-api-server
      user: tenant-user
    name: capsule-context
current-context: capsule-context
users:
  - name: tenant-user
    user:
      exec:
        apiVersion: client.authentication.k8s.io/v1beta1
        args:
          - get-token
          - --login
          - azurecli
          - --server-id
          - 6dae42f8-4368-4678-94ff-3960e28e3630 # <-- well-known server-id for Azure AKS service
        command: kubelogin
EOT

export KUBECONFIG=/tmp/bob/kubeconfig.yaml
k config set-context --current --namespace=app-bob-default
```


# DEMO

## authenticated users

```shell
# get logged in user
k auth whoami


```

## get ns

```shell
kubectl get namespaces
kubectl create namespace name-not-compliant
kubectl create namespace app-alice-ns2
kubectl create namespace app-alice-ns3
kubectl create namespace app-alice-ns4 
```

## enriched data 

```shell
kubectl describe namespaces app-alice-ns2
```


## network policies

```shell
kubectl get -n app-alice-default networkpolicies
kubectl get -n app-alice-default networkpolicies -o yaml
```

## create pod 

```shell
kubectl run -n app-alice-default whoami --image containous/whoami
kubectl get pod
kubectl describe pod whoami
```

## deploy helm with ingress

```shell 
helm upgrade --install --create-namespace --namespace app-alice-default echo-server /Users/gchiesa/git/helm-echo-server
kubectl get pod
```

switch to bob

```shell
helm upgrade --install --create-namespace --namespace app-bob-default echo-server /Users/gchiesa/git/helm-echo-server
```