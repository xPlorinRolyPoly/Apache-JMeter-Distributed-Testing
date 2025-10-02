# <img src="../../icons/vault.png" alt= “minikube” width="25" height="25"> Injecting Secrets into Kubernetes Pods via Vault Agent Containers
Deploying applications that act as secret consumers of Vault require the application to:
* Authenticate and acquire a client token.
* Manage the lifecycle of the token.
* Retrieve secrets from Vault.
* Manage the leases of any dynamic secrets.

Vault Agent takes responsibility for these tasks and enables your applications to remain unaware of Vault. 
However, this introduces a new requirement that deployments install and configure Vault Agent alongside the application as a sidecar.

For more information please read [Hashicorp Vault Official Doc](https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-sidecar).

## Install Vault using Helm chart <img src="../../icons/helm.svg" alt= “minikube” width="22" height="22">
In Minikube k8s cluster, switch to namespace `default` and then run below commands to install vault.

* Run below command to add the HashiCorp Helm repository and update it:
  ```shell
  helm repo add hashicorp https://helm.releases.hashicorp.com
  helm repo update
  ```

* Run below command to install the latest version of the Vault server running in development mode:
  ```shell
  export REPO_DIR="<path\to\apache-jmeter-distributed-testing>"
  cd "$REPO_DIR\k8s\vault"
  helm install vault hashicorp/vault --namespace default -f override-values.yaml
  ```
  With the above command, vault pod and vault agent injector pod are deployed in the `default` namespace.

* Run below command to port-forward the vault service, so vault UI can be accessed at localhost:
  ```shell
  kubectl port-forward -n default services/vault 8200:8200
  ```
  With above command you can access the Vault UI on http://localhost:8200/ 
  On Login page, select Method `Token` and enter value `vault@alpana` in field Token.

## Configure Secrets in Vault
* Start an interactive shell session on the `vault-0` pod, by running below command:
  ```shell
  kubectl exec -n default -it vault-0 -- sh
  ```

* Enable kv-v2 secrets at the path `nft`, by running below command inside `vault-0` container:
  ```shell
  vault secrets enable -path=nft kv-v2
  ```

* Configure secrets in path `nft/test-automation/prod/jmeter-dev`, by running below commands inside `vault-0` container:
  ```shell
  vault kv put nft/test-automation/prod/jmeter-dev API_USERNAME="alpana" API_PASSWORD="httpbin"
  vault kv patch nft/test-automation/prod/jmeter-dev INFLUX_TOKEN="ScHCbdo6A2goQ9afT3iGnh4VoZUvViEwmH8dpcGWm45B2mfAN2n1EM33oG5otv2cCTvkiO92-CFpC9wHpQXNVQ=="
  ```

* Verify that the secret is defined at the path `nft/test-automation/prod/jmeter-dev`, by running below command inside `vault-0` container:
  ```shell
  vault kv get nft/test-automation/prod/jmeter-dev
  ```

## Configure Kubernetes Authentication
* Start an interactive shell session on the `vault-0` pod, by running below command:
  ```shell
  kubectl exec -n default -it vault-0 -- sh
  ```

* Enable the Kubernetes authentication method, by running below command inside `vault-0` container:
  ```shell
  vault auth enable -path=kubernetes-nft-test-automation-dev kubernetes
  ```

* Configure the Kubernetes authentication method to use the location of the minikube Kubernetes API, by running below command inside `vault-0` container:
  ```shell
  vault write auth/kubernetes-nft-test-automation-dev/config \
    kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"
  ```

* Write out the policy named `nft-test-automation-dev` that enables the read capability for secrets at path `nft/test-automation/prod/jmeter-dev`, by running below command inside `vault-0` container:
  ```shell
  vault policy write nft-test-automation-dev - <<EOF
  path "nft/data/test-automation/prod/jmeter-dev" {
    capabilities = ["read"]
  }
  
  path "nft/data/test-automation/prod/jmeter-dev/*" {
    capabilities = ["read"]
  }
  EOF
  ```

* Create a Kubernetes authentication role named `reader`, by running below command inside `vault-0` container:
  ```shell
  vault write auth/kubernetes-nft-test-automation-dev/role/reader \
    bound_service_account_names=sa-test-automation \
    bound_service_account_namespaces=ns-test-automation-jmeter-dev \
    policies=nft-test-automation-dev \
    ttl=24h
  ```

## Storage
Since vault is deployed in development mode, vault runs in-memory and starts unsealed. No other secret backend is 
supported in `-dev` mode.

---
> **Warning**
> 
> All secretes and configurations of Vault will be deleted after a restart, because vault is deployed in development mode.
---