# DevOps AI Playbook — Command Reference

Complete log of commands used in this session, grouped by phase, with example outputs.

---

## 1. Project Setup

### Navigate to project
```bash
cd projects/boutique-microservices
```

### Install dependencies
```bash
npm install
```
**Output:**
```
added 1516 packages, and audited 1524 packages in 5m
28 vulnerabilities (9 low, 5 moderate, 14 high)
```

### Build frontend and backend
```bash
npm run build
```
**Output:**
```
> boutique-microservices@1.0.0 build:frontend
Creating an optimized production build...
Compiled with warnings.
File sizes after gzip:
  227.51 kB  build/static/js/main.19a9e1c2.js

> gateway-service@1.0.0 build
> tsc
... (all 5 backend services compiled)
```

---

## 2. Docker — Local Environment

### Start all services
```bash
docker compose up -d --build
```
**Output:**
```
Container boutique-postgres    Started
Container boutique-auth        Started
Container boutique-gateway     Started
Container boutique-products    Started
Container boutique-orders      Started
Container boutique-frontend    Started
Container boutique-prometheus  Started
Container boutique-grafana     Started
```

### Check running containers
```bash
docker ps
```
**Output:**
```
CONTAINER ID   IMAGE                    PORTS                    NAMES
628eafe23250   grafana/grafana:latest   0.0.0.0:3007->3000/tcp   boutique-grafana
1d68d86c4e0e   prom/prometheus:latest   0.0.0.0:9090->9090/tcp   boutique-prometheus
```

### Stop all containers
```bash
docker compose down
```
**Output:**
```
Container boutique-frontend   Removed
Container boutique-gateway    Removed
Container boutique-postgres   Removed
Network boutique-microservices_boutique-network  Removed
```

---

## 3. AWS Authentication

### Verify AWS credentials
```bash
aws sts get-caller-identity
```
**Output:**
```json
{
    "UserId": "AIDA33LJWAUK2M6YVZCO5",
    "Account": "814654817557",
    "Arn": "arn:aws:iam::814654817557:user/anup"
}
```

---

## 4. Terraform — Infrastructure

### Navigate to infrastructure
```bash
cd projects/Infrastructure
```

### Initialize Terraform
```bash
terraform init
```
**Output:**
```
Initializing modules...
- argocd in modules/argocd
- ecr in modules/ecr
- vpc in modules/vpc
- eks in modules/eks
Terraform has been successfully initialized!
```

### Preview infrastructure changes
```bash
terraform plan
```
**Output:**
```
Plan: 32 to add, 0 to change, 0 to destroy.
Changes to Outputs:
  + cluster_endpoint = (known after apply)
  + cluster_name     = "eks-cluster"
  + ecr_urls         = { auth, frontend, gateway, order-service, orders, product-service, user-service }
```

### Apply infrastructure
```bash
terraform apply --auto-approve
```
**Output:**
```
Apply complete! Resources: 32 added, 0 changed, 0 destroyed.

Outputs:
cluster_endpoint = "https://B86004A3150DDA84D6637D3876153FD2.gr7.us-east-1.eks.amazonaws.com"
cluster_name     = "eks-cluster"
ecr_urls = {
  "auth"            = "814654817557.dkr.ecr.us-east-1.amazonaws.com/auth"
  "frontend"        = "814654817557.dkr.ecr.us-east-1.amazonaws.com/frontend"
  "gateway"         = "814654817557.dkr.ecr.us-east-1.amazonaws.com/gateway"
  "order-service"   = "814654817557.dkr.ecr.us-east-1.amazonaws.com/order-service"
  "orders"          = "814654817557.dkr.ecr.us-east-1.amazonaws.com/orders"
  "product-service" = "814654817557.dkr.ecr.us-east-1.amazonaws.com/product-service"
  "user-service"    = "814654817557.dkr.ecr.us-east-1.amazonaws.com/user-service"
}
```

### Show current state
```bash
terraform show
```
**Output (key resources):**
```
module.eks.aws_eks_cluster.eks:
  id       = "eks-cluster"
  status   = "ACTIVE"
  endpoint = "https://B86004A3150DDA84D6637D3876153FD2.gr7.us-east-1.eks.amazonaws.com"
  vpc_id   = "vpc-04e4c84c95855f1d3"

module.argocd.helm_release.argocd:
  status = "deployed"
```

---

## 5. Kubernetes — Cluster Setup

### Configure kubectl for EKS
```bash
aws eks update-kubeconfig --region us-east-1 --name eks-cluster
```
**Output:**
```
Added new context arn:aws:eks:us-east-1:814654817557:cluster/eks-cluster to /Users/anup/.kube/config
```

### List nodes
```bash
kubectl get nodes
```
**Output:**
```
NAME                        STATUS   ROLES    AGE   VERSION
ip-10-1-1-73.ec2.internal   Ready    <none>   60m   v1.34.7-eks-40737a8
```

### List all pods across all namespaces
```bash
kubectl get pods -A
```
**Output:**
```
NAMESPACE     NAME                                                        READY   STATUS    RESTARTS   AGE
argocd        argocd-application-controller-0                             1/1     Running   0          3h35m
argocd        argocd-server-7696b9bcf5-krqpr                              1/1     Running   0          3h35m
boutique      auth-6649756745-jzc72                                       1/1     Running   0          32m
boutique      boutique-postgres-0                                         1/1     Running   0          116m
boutique      frontend-6747f4cd87-k54kw                                   1/1     Running   0          32m
boutique      gateway-56c56f8b98-4dfsv                                    1/1     Running   0          32m
boutique      order-service-6fdfb8bd69-p9lwq                              1/1     Running   0          32m
boutique      orders-99c77cbf5-zbrr5                                      1/1     Running   0          32m
boutique      product-service-5b98df5b64-qdlkp                            1/1     Running   0          32m
boutique      user-service-7499ddcb84-q262c                               1/1     Running   0          32m
monitoring    kube-prometheus-stack-grafana-56cfc7b686-b6nrr              3/3     Running   0          3h12m
monitoring    prometheus-kube-prometheus-stack-prometheus-0               2/2     Running   0          3h12m
```

---

## 6. Kubernetes — Boutique App Deployment

### Apply all manifests via Kustomize
```bash
kubectl apply -k gitops/
```
**Output:**
```
namespace/boutique created
configmap/boutique-db-dump created
secret/boutique-secrets created
service/auth created
service/gateway created
service/frontend created
deployment.apps/auth created
deployment.apps/frontend created
deployment.apps/gateway created
deployment.apps/order-service created
deployment.apps/orders created
deployment.apps/product-service created
deployment.apps/user-service created
statefulset.apps/boutique-postgres created
servicemonitor.monitoring.coreos.com/boutique-services created
```

### List pods in boutique namespace
```bash
kubectl get pods -n boutique
```
**Output:**
```
NAME                               READY   STATUS      RESTARTS   AGE
auth-6649756745-jzc72              1/1     Running     0          31m
boutique-db-restore-qbkmv          0/1     Completed   0          36m
boutique-postgres-0                1/1     Running     0          114m
frontend-6747f4cd87-k54kw          1/1     Running     0          31m
gateway-56c56f8b98-4dfsv           1/1     Running     0          31m
order-service-6fdfb8bd69-p9lwq     1/1     Running     0          31m
orders-99c77cbf5-jmb4f             1/1     Running     0          6m
product-service-5b98df5b64-4h9m7   1/1     Running     0          2m
user-service-7499ddcb84-q262c      1/1     Running     0          31m
```

### List pods with full details
```bash
kubectl get pods -n boutique -o wide
```
**Output:**
```
NAME                               READY   STATUS    IP           NODE
auth-6649756745-jzc72              1/1     Running   10.1.1.162   ip-10-1-1-73.ec2.internal
boutique-postgres-0                1/1     Running   10.1.1.216   ip-10-1-1-73.ec2.internal
frontend-6747f4cd87-k54kw          1/1     Running   10.1.1.229   ip-10-1-1-73.ec2.internal
gateway-56c56f8b98-4dfsv           1/1     Running   10.1.1.204   ip-10-1-1-73.ec2.internal
```

### Describe all pods
```bash
kubectl describe pods -n boutique
```
**Output (key fields per pod):**
```
Name:   auth-6649756745-jzc72
Status: Running
Image:  814654817557.dkr.ecr.us-east-1.amazonaws.com/auth:6b6f4e89c8ed...
Port:   3002/TCP
Started: Sun, 03 May 2026 13:51:30 +0530
Ready:  True
Restart Count: 0
```

### List services
```bash
kubectl get svc -n boutique
```
**Output:**
```
NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
auth                ClusterIP   172.20.98.6      <none>        3002/TCP   95m
boutique-postgres   ClusterIP   None             <none>        5432/TCP   95m
frontend            ClusterIP   172.20.250.175   <none>        3000/TCP   95m
gateway             ClusterIP   172.20.248.79    <none>        3001/TCP   95m
order-service       ClusterIP   172.20.25.85     <none>        3002/TCP   95m
orders              ClusterIP   172.20.56.221    <none>        3005/TCP   95m
product-service     ClusterIP   172.20.2.36      <none>        3003/TCP   95m
user-service        ClusterIP   172.20.30.96     <none>        3006/TCP   95m
```

### List all deployments with details
```bash
kubectl get deployments -A -o wide
```
**Output:**
```
NAMESPACE   NAME              READY   UP-TO-DATE   AVAILABLE   AGE
boutique    auth              1/1     1            1           10h
boutique    frontend          1/1     1            1           10h
boutique    gateway           1/1     1            1           10h
boutique    order-service     1/1     1            1           10h
boutique    orders            1/1     1            1           9h
boutique    product-service   1/1     1            1           9h
boutique    user-service      1/1     1            1           10h
argocd      argocd-server     1/1     1            1           12h
```

### Apply database restore job
```bash
kubectl apply -f gitops/k8s/database/restore-job.yml
```
**Output:**
```
job.batch/boutique-db-restore created
```

### Check restore job logs
```bash
kubectl logs boutique-db-restore-qbkmv -n boutique
```
**Output:**
```
Waiting for Postgres...
boutique-postgres:5432 - accepting connections
Restoring dump...
CREATE DATABASE
CREATE TABLE
COPY 5
Restore completed.
```

### Verify databases created
```bash
kubectl exec boutique-postgres-0 -n boutique -- psql -U postgres -c "\l"
```
**Output:**
```
    Name     |  Owner
-------------+----------
 auth_db     | postgres
 boutique_db | postgres
 orders_db   | postgres
 products_db | postgres
 users_db    | postgres
```

---

## 7. ArgoCD — GitOps

### Deploy ArgoCD application
```bash
kubectl apply -f gitops/argo-cd.yml -n argocd
```
**Output:**
```
application.argoproj.io/boutique created
```

### Get ArgoCD admin password
```bash
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
```
**Output:**
```
hdZlpKvZxNon2EDb
```

### Port-forward ArgoCD UI
```bash
kubectl port-forward svc/argocd-server -n argocd 9090:80
```
Then open: **http://localhost:9090** (admin / hdZlpKvZxNon2EDb)

### List ArgoCD pods
```bash
kubectl get pods -n argocd
```
**Output:**
```
NAME                                                READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                     1/1     Running   0          105m
argocd-applicationset-controller-67d88d578b-899fv   1/1     Running   0          105m
argocd-dex-server-687b796d67-2w2hr                  1/1     Running   0          105m
argocd-notifications-controller-7ccf5fbd9d-mdndz    1/1     Running   0          105m
argocd-redis-66cb974645-ts76d                       1/1     Running   0          105m
argocd-repo-server-6db8c546f-qf8sd                  1/1     Running   0          105m
argocd-server-7696b9bcf5-krqpr                      1/1     Running   0          105m
```

### Scale a deployment to 0 (GitOps drift test)
```bash
kubectl scale deployment orders -n boutique --replicas=0
```
**Output:**
```
deployment.apps/orders scaled
```
> ArgoCD self-heals and restores to 1 replica within ~30 seconds (when `selfHeal: true`)

### Delete a deployment (GitOps drift test)
```bash
kubectl delete deployment product-service -n boutique
```
**Output:**
```
deployment.apps "product-service" deleted
```
> ArgoCD recreates it automatically from Git

---

## 8. Monitoring — Prometheus & Grafana

### List monitoring pods
```bash
kubectl get pods -n monitoring
```
**Output:**
```
NAME                                                        READY   STATUS    AGE
alertmanager-kube-prometheus-stack-alertmanager-0           2/2     Running   3h17m
kube-prometheus-stack-grafana-56cfc7b686-b6nrr              3/3     Running   3h17m
kube-prometheus-stack-kube-state-metrics-57994bbc9b-b7p95   1/1     Running   3h17m
kube-prometheus-stack-operator-77bf8fff64-bzfbq             1/1     Running   3h17m
kube-prometheus-stack-prometheus-node-exporter-k6l8k        1/1     Running   3h17m
prometheus-kube-prometheus-stack-prometheus-0               2/2     Running   3h17m
```

### List monitoring services
```bash
kubectl get svc -n monitoring
```
**Output:**
```
NAME                                    TYPE        CLUSTER-IP       PORT(S)
kube-prometheus-stack-grafana           ClusterIP   172.20.244.127   80/TCP
kube-prometheus-stack-prometheus        ClusterIP   172.20.172.205   9090/TCP
kube-prometheus-stack-alertmanager      ClusterIP   172.20.198.125   9093/TCP
```

### Expose Prometheus via LoadBalancer
```bash
kubectl patch svc kube-prometheus-stack-prometheus -n monitoring \
  -p '{"spec": {"type": "LoadBalancer"}}'
```
**Output:**
```
NAME                               TYPE           EXTERNAL-IP
kube-prometheus-stack-prometheus   LoadBalancer   ab8340e63a9fc4bb590655c640faa240-1193042091.us-east-1.elb.amazonaws.com
```
Access: **http://ab8340e63a9fc4bb590655c640faa240-1193042091.us-east-1.elb.amazonaws.com:9090**

### Port-forward Grafana
```bash
kubectl port-forward svc/kube-prometheus-stack-grafana -n monitoring 3030:80
```
Access: **http://localhost:3030** (admin / prom-operator)

### Port-forward Prometheus
```bash
kubectl port-forward svc/kube-prometheus-stack-prometheus -n monitoring 9091:9090
```
Access: **http://localhost:9091**

---

## 9. Logging — Fluent Bit to CloudWatch

### Add AWS Helm repo
```bash
helm repo add aws https://aws.github.io/eks-charts
helm repo update
```

### Install Fluent Bit
```bash
helm upgrade --install aws-for-fluent-bit aws/aws-for-fluent-bit \
  --namespace amazon-cloudwatch \
  --create-namespace \
  --set cloudWatch.enabled=true \
  --set cloudWatch.region=us-east-1 \
  --set cloudWatch.logGroupName=/eks/boutique/pods \
  --set cloudWatch.logStreamPrefix=from-fluent-bit- \
  --set firehose.enabled=false \
  --set kinesis.enabled=false \
  --set elasticsearch.enabled=false
```
**Output:**
```
Release "aws-for-fluent-bit" does not exist. Installing it now.
STATUS: deployed
DESCRIPTION: Install complete
```

### Attach CloudWatch IAM policy to node role
```bash
aws iam attach-role-policy \
  --role-name eks-cluster-node-role \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchLogsFullAccess
```

### Fix IMDS hop limit for pods
```bash
aws ec2 modify-instance-metadata-options \
  --instance-id i-0c8fddd461ebe560b \
  --http-put-response-hop-limit 2 \
  --http-tokens required \
  --region us-east-1
```

### Restart Fluent Bit after IAM fix
```bash
kubectl rollout restart daemonset/aws-for-fluent-bit -n amazon-cloudwatch
```
**CloudWatch Log Group:** `/eks/boutique/pods`

---

## 10. CI/CD — GitHub Actions

### Trigger CI pipeline
```bash
git commit --allow-empty -m "start CI pipeline"
git push origin main
```

### Check pipeline status
```bash
gh run list --repo AnupCloud/devops-ai-playbook --limit 5
```
**Output:**
```
completed  success  ci: trigger pipeline  Boutique CI Pipeline  main  push  2m28s
```

### List GitHub secrets
```bash
gh secret list --repo AnupCloud/devops-ai-playbook
```
**Output:**
```
AWS_ACCESS_KEY_ID      2026-05-03T06:31:08Z
AWS_ACCOUNT_ID         2026-05-03T06:34:13Z
AWS_REGION             2026-05-03T06:35:53Z
AWS_SECRET_ACCESS_KEY  2026-05-03T06:31:52Z
```

---

## 11. AIOps Assistant — Bedrock Agent

### Navigate to AIOps project
```bash
cd projects/aiops-assistant
```

### Install dependencies
```bash
pip install -r requirements.txt
```

### Deploy Bedrock Agent
```bash
./deploy.sh
```
**Output:**
```
[0/3] Pre-flight checks...
  ✓ Lambda: aiops-fetch-logs
  ✓ Lambda: aiops-fetch-metrics
  ✓ Lambda: aiops-fetch-health
  ✓ IAM role: aiops-bedrock-agent-role

[1/3] Configuring Lambda functions...
  ✓ aiops-fetch-logs timeout set to 30s

[2/3] Creating Bedrock Agent: aiops-assistant...
  ✓ Agent created: 3OEGBZXZM7

[3/3] Adding action groups and preparing agent...
  ✓ fetch_logs
  ✓ fetch_metrics
  ✓ fetch_service_health

Agent ID : 3OEGBZXZM7
Region   : us-east-1
```

### Create Bedrock Agent alias
```bash
aws bedrock-agent create-agent-alias \
  --agent-id 3OEGBZXZM7 \
  --agent-alias-name "production" \
  --region us-east-1 \
  --query 'agentAlias.agentAliasId' --output text
```
**Output:**
```
QKUTHI8DGQ
```

### Run Streamlit UI
```bash
streamlit run app.py
```
Access: **http://localhost:8501**

---

## Port Reference

| Service | Local Port | Command |
|---|---|---|
| Frontend | 3005 | `kubectl port-forward svc/frontend -n boutique 3005:3000` |
| Gateway API | 3001 | `kubectl port-forward svc/gateway -n boutique 3001:3001` |
| ArgoCD UI | 9090 | `kubectl port-forward svc/argocd-server -n argocd 9090:80` |
| Grafana | 3030 | `kubectl port-forward svc/kube-prometheus-stack-grafana -n monitoring 3030:80` |
| Prometheus | 9091 | `kubectl port-forward svc/kube-prometheus-stack-prometheus -n monitoring 9091:9090` |
| Streamlit (AIOps) | 8501 | `streamlit run app.py` |

---

## Key Resource IDs

| Resource | Value |
|---|---|
| AWS Account | `814654817557` |
| EKS Cluster | `eks-cluster` |
| EKS Endpoint | `https://B86004A3150DDA84D6637D3876153FD2.gr7.us-east-1.eks.amazonaws.com` |
| EC2 Node | `i-0c8fddd461ebe560b` |
| VPC | `vpc-04e4c84c95855f1d3` |
| Bedrock Agent ID | `3OEGBZXZM7` |
| Bedrock Alias ID | `QKUTHI8DGQ` |
| ECR Registry | `814654817557.dkr.ecr.us-east-1.amazonaws.com` |
| GitHub Repo | `https://github.com/AnupCloud/devops-ai-playbook` |
