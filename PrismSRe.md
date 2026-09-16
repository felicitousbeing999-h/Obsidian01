![Pasted image 20260816011206.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260816011206.png)rpull' role assignment under scope '/subscriptions/94321bdf-da5f-4cc4-86eb-fe1c24335bfb/resourceGroups/k8sgpt/providers/Microsoft.ContainerRegistry/registries/hardikarora'
The output includes credentials that you must protect. Be sure that you do not include these credentials in your code or check the credentials into your source control. For more information, see https://aka.ms/azadsp-cli

# this is app id
{
  "appId": "ce805848-e000-4bc8-84da-481aebfbb732",
  "displayName": "prismsre-acr-pull",
  "password": "XJ28Q~8f0pV~-CsghvCyDIjSpqYXxGWqi3TK3b~D",
  "tenant": "c0177927-d841-4ce1-ae7d-c272a34a3b20"
}

deploy-prismsre.sh




```shell

#!/usr/bin/env bash
set -euo pipefail

echo "=========================================="
echo "       PrismSRE ACR Deployment"
echo "=========================================="
echo

# ---------- Configuration ----------
ACR_SERVER="hardikarora.azurecr.io"
IMAGE="hardikarora.azurecr.io/prismsre:3"
NAMESPACE="default"

# ---------- Prerequisite checks ----------
command -v kubectl >/dev/null || {
    echo "ERROR: kubectl is not installed."
    exit 1
}

echo "Kubernetes cluster:"
kubectl cluster-info >/dev/null

# ---------- Credentials ----------
echo
echo "Enter the Service Principal credentials"
echo "This identity should have ONLY AcrPull on the ACR."
echo

read -rp "Service Principal Client ID: " SP_USERNAME
read -rsp "Service Principal Password: " SP_PASSWORD
echo

echo
echo "Enter the Gemini API key."
read -rsp "Gemini API Key: " GEMINI_API_KEY
echo

# ---------- Create ACR image pull secret ----------
echo
echo "[1/4] Creating ACR image pull secret..."

kubectl create secret docker-registry acr-pull-secret \
    --namespace "$NAMESPACE" \
    --docker-server="$ACR_SERVER" \
    --docker-username="$SP_USERNAME" \
    --docker-password="$SP_PASSWORD" \
    --dry-run=client -o yaml |
kubectl apply -f -

echo "✓ acr-pull-secret created"

# ---------- Create Gemini secret ----------
echo
echo "[2/4] Creating Gemini API secret..."

kubectl create secret generic kubeops-ai-secret \
    --namespace "$NAMESPACE" \
    --from-literal=GOOGLE_API_KEY="$GEMINI_API_KEY" \
    --dry-run=client -o yaml |
kubectl apply -f -

echo "✓ kubeops-ai-secret created"

# ---------- Deploy PrismSRE ----------
echo
echo "[3/4] Deploying PrismSRE..."

cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prismsre
  namespace: $NAMESPACE
spec:
  replicas: 1

  selector:
    matchLabels:
      app: prismsre

  template:
    metadata:
      labels:
        app: prismsre

    spec:
      imagePullSecrets:
        - name: acr-pull-secret

      containers:
        - name: prismsre
          image: $IMAGE
          imagePullPolicy: IfNotPresent

          ports:
            - containerPort: 8000

          envFrom:
            - secretRef:
                name: kubeops-ai-secret

---
apiVersion: v1
kind: Service
metadata:
  name: prismsre-service
  namespace: $NAMESPACE
spec:
  type: ClusterIP

  selector:
    app: prismsre

  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
EOF

echo "✓ PrismSRE deployment applied"

# ---------- Verify ----------
echo
echo "[4/4] Waiting for PrismSRE pod..."

kubectl rollout status deployment/prismsre \
    --namespace "$NAMESPACE" \
    --timeout=180s

echo
echo "=========================================="
echo "          Deployment Successful"
echo "=========================================="
echo

kubectl get deployment prismsre
kubectl get pods -l app=prismsre
kubectl get service prismsre-service

echo
echo "Image:"
echo "  $IMAGE"

echo
echo "To inspect the pod:"
echo "  kubectl describe pod -l app=prismsre"

echo
echo "To view logs:"
echo "  kubectl logs -l app=prismsre"

echo
echo "To verify the pulled image:"
echo "  kubectl get pod -l app=prismsre \\"
echo "    -o jsonpath='{.items[0].status.containerStatuses[0].imageID}'"

echo
echo "Done."

```

something