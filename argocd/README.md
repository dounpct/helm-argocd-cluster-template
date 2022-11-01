# Add ArgoCD helm repo

    helm repo add argo-cd https://argoproj.github.io/argo-helm
    helm dep update

# Install ArgoCD vault plugin

    curl -Lo argocd-vault-plugin https://github.com/argoproj-labs/argocd-vault-plugin/releases/download/v1.12.0/argocd-vault-plugin_1.12.0_linux_amd64
    chmod +x argocd-vault-plugin
    sudo mv argocd-vault-plugin /usr/local/bin

# Render Secret from GCP Secret Manager

    export GOOGLE_APPLICATION_CREDENTIALS="/mnt/d/firework/gcr-authen-json/gcp-dmp-devops.json"
    export AVP_TYPE=gcpsecretmanager

    argocd-vault-plugin generate ./
    argocd-vault-plugin generate ./values/test.yaml

    helm template argocd --include-crds -n argocd . -f values/values-argocd.yaml | argocd-vault-plugin generate -
    helm template argocd --include-crds -n argocd . -f values/values-argocd.yaml | argocd-vault-plugin generate - | kubectl -n argocd apply -f -

### first time access
    kubectl port-forward svc/argocd-server -n argocd  3333:80

### admin secret
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
    sync all argocd start application
    open fw on gcp
    access with url

# for rollout after change config of Dex

    kubectl rollout restart deploy/argocd-server -n argocd

# Create Local User

Add the following key/value under argo-cd.server.config

  accounts.ACCOUNT_NAME: apiKey

To generate the token login as admin and run the following command

  // LOGIN
  argocd login YOUR_ARGO_NAME.domain.local:443 --sso

  // GENERATE THE TOKEN
  argocd account generate-token --account ACCOUNT_NAME

Replace ACCOUNT_NAME with your creating account name.


# note for helm render

    helm template argocd --include-crds -n argocd . -f values/values-argocd-bncloud.yaml | grep "# Source"

    helm template argocd --include-crds -n argocd . -f values/values-argocd-bncloud.yaml -s templates/istio.yaml

    helm template argocd --include-crds -n argocd . -f values/values-argocd-bncloud.yaml -s charts/argo-cd/templates/argocd-configs/argocd-secret.yaml | argocd-vault-plugin generate -

    helm template argocd --include-crds -n argocd . -f values/values-argocd-bncloud.yaml -s charts/argo-cd/templates/argocd-repo-server/deployment.yaml | argocd-vault-plugin generate -

    helm template argocd --include-crds -n argocd . -f values/values-argocd-bncloud.yaml -s templates/secret.yaml | argocd-vault-plugin generate -