# Vault Agent Injector Example

https://learn.hashicorp.com/vault/getting-started-k8s/sidecar#apply-a-template-to-the-injected-secrets

```bash
git clone https://github.com/hashicorp/vault-helm.git
helm3 install vault ./vault-helm --set "server.dev.enabled=true"
or
helm3 install vault ./vault-helm --set "server.ha.enabled=true"
kubectl get pods
kubectl exec -it vault-0 /bin/sh
vault auth enable userpass
vault write auth/userpass/users/testuser password=kubernauts policies=admins
kubectl port-forward vault-0 8200:8200
vault secrets enable -path=internal kv-v2
vault kv put internal/database/config username="db-readonly-username" password="db-secret-password"
vault kv get internal/database/config
vault auth enable kubernetes

vault write auth/kubernetes/config \
         token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
         kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443" \
         kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt

vault policy write internal-app - <<EOF
path "internal/data/database/config" {
capabilities = ["read"]
}
EOF

vault write auth/kubernetes/role/internal-app \
        bound_service_account_names=internal-app \
        bound_service_account_namespaces=default \
        policies=internal-app \
        ttl=24h

kubectl get serviceaccounts
k create sa internal-app --dry-run -o yaml > service-account-internal-app.yaml
cat service-account-internal-app.yaml
k create -f service-account-internal-app.yaml
k create -f deployment-01-orgchart.yml
k get pods

The Vault-Agent injector looks for deployments that define specific annotations. None of these annotations exist within the current deployment. This means that no secrets are present on the orgchart container within the orgchart-69697d9598-l878s pod.

Verify that no secrets are written to the orgchart container in the orgchart-69697d9598-l878s pod.

k exec orgchart-69697d9598-p7wfc -c orgchart -- ls /vault/secrets

Inject secrets into the pod

```
