# Argocd_Installation
Argocd_Installation

Helm command to install ArgoCD;

helm install -f values.yaml banct-argo-cd argo/argo-cd --version 7.6.8 -n argo-cd


Changes done on values.yaml;

1) global:
  # -- Default domain used by all components
  ## Used for ingresses, certificates, SSO, notifications, etc.
  domain: argocd-app.com

/////////

2) crds:
  # -- Install and upgrade CRDs
  install: false

///////////

3)   ingress:
    # -- Enable an ingress resource for the Argo CD server
    enabled: true
    # -- Specific implementation for ingress controller. One of `generic`, `aws` or `gke`
    ## Additional configuration might be required in related configuration sections
    controller: generic
    # -- Additional ingress labels
    labels: {}
    # -- Additional ingress annotations
    ## Ref: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-1-ssl-passthrough
    annotations:
      alb.ingress.kubernetes.io/scheme: internet-facing  # Change to "internal" for internal ALB
      alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80}, {"HTTPS":443}]'
      alb.ingress.kubernetes.io/secure-listener-redirect: 'true'
      alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
      alb.ingress.kubernetes.io/backend-protocol: HTTPS
      alb.ingress.kubernetes.io/target-type: ip  # Use IP mode
      alb.ingress.kubernetes.io/inbound-cidrs: 0.0.0.0/0  # Allow traffic from anywhere (adjust as needed)
      alb.ingress.kubernetfalsees.io/subnets: subnet-xx,subnet-xx
      alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:eu-west-2:xxxxxxxxxxxxx:certificate/xxxxxx-xxxxx-xxx-xxx-xxxx
    ingressClassName: "alb"

    # -- Argo CD server hostname
    # @default -- `""` (defaults to global.domain)
    hostname: "argocd-app.com"

4) Sonra aws de route 53, ACM, ALB, TG ayarlarını yapıp, domaine erişimi sağladım. Sonrada websitesine erişim username : admin, password u bulmaya çalıştım. Aşağıdaki komut ile buldum

5) Argocd yi indirdikten sonra Gitlab ile entegre edebilmek için gitlab ci yaml ı güncelledim. Güncellerken argocd de token oluşturmam gerekti. Bunun için argo cm ye token oluşturma yetkisi vermek için "accounts.admin: apiKey, login" satırını ekledim. Sonra da aşağıdaki gibi token oluşturdum sonra bu tokenı gitlab ci yaml a yazdım

# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  accounts.admin: apiKey, login
  admin.enabled: "true"
  application.instanceLabelKey: argocd.argoproj.io/instance
  exec.enabled: "false"
  server.rbac.log.enforce.enable: "false"
  statusbadge.enabled: "false"
  timeout.hard.reconciliation: 0s
  timeout.reconciliation: 180s
  url: https://argocd.banct-app.com
kind: ConfigMap
metadata:
  annotations:
    meta.helm.sh/release-name: banct-argo-cd
    meta.helm.sh/release-namespace: argo-cd
  creationTimestamp: "2024-10-07T06:55:49Z"
  labels:
    app.kubernetes.io/component: server
    app.kubernetes.io/instance: banct-argo-cd
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: argocd-cm
    app.kubernetes.io/part-of: argocd
    app.kubernetes.io/version: v2.12.4
    helm.sh/chart: argo-cd-7.6.8
  name: argocd-cm
  namespace: argo-cd
  resourceVersion: "742896086"
  uid: 190e9402-a2ad-4a62-ade9-12316b91fde6


dell@dell-Latitude-3420:~$ argocd account generate-token --account admin --grpc-web
FATA[0010] rpc error: code = Unknown desc = POST https://argocd.banct-app.com/account.AccountService/CreateToken failed with status code 504 
dell@dell-Latitude-3420:~$ argocd login argocd.banct-app.com --username admin --password nTaKudIhQUG-gv3W  --insecure
WARN[0000] Failed to invoke grpc call. Use flag --grpc-web in grpc calls. To avoid this warning message, use flag --grpc-web. 
'admin:login' logged in successfully
Context 'argocd.banct-app.com' updated
dell@dell-Latitude-3420:~$ argocd account generate-token --account admin --grpc-web
eyJhbxxxx80GRxxxx85tqa-TMG6puIf5-b0


6) Argo web password unu öğrenmek için

dell@dell-Latitude-3420:~$ kubectl get secret -n argo-cd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
xxxx-xx


7) Then we needed to add repo and create app "northdraw-svc" on argocd. To do that, we first needed to create personal access token on gitlab to use it in below command as glpat-xxx


dell@dell-Latitude-3420:~/Downloads/argo-cd-7.6.8/argo-cd$ argocd repo add https://gitlab.com/b5346/b-devops.git \
    --username melike.ozen@b.com \
    --password glpat-xxx
Repository 'https://gitlab.com/bxx/b-devops.git' added
&&&&&&&&&&

dell@dell-Latitude-3420:~/Downloads/argo-cd-7.6.8/argo-cd$ argocd app create northrow-svc \
    --repo https://gitlab.com/bxx/b-devops.git \
    --path helm-deployment/api-helm \
    --dest-server https://kubernetes.default.svc \
    --dest-namespace default \
    --revision main \
    --sync-policy automated
application 'northrow-svc' created

8) Sonra argocd stage eklemek için gitlab ci yaml ı güncelleyebiliriz.

App sync i lokalde test ettim

dell@dell-Latitude-3420:~/Downloads/argo-cd-7.6.8/argo-cd$ argocd app sync banct-northrow-integration-svc --auth-token eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJhcmdvY2QiLCJzdWIiOiJhZG1pbjphcGlLZXkiLCJuYmYiOjE3Mjg1ODc4NDksImlhdCI6MTcyODU4Nzg0OSwianRpIjoiZTg5OTdmZDEtYTEwOC00M2UwLThlYjgtNTVjMzcwOTJmYWIyIn0.Q4bbrnUZK_fkTCWsz2taYdNN1GnoP2aYrjXhXPsKKEc --grpc-web --insecure
TIMESTAMP                  GROUP        KIND   NAMESPACE                  NAME                      STATUS   HEALTH         HOOK  MESSAGE
2024-10-11T01:30:43+03:00            Service     default  banct-northrow-integration-svc-service    Synced  Healthy               
2024-10-11T01:30:43+03:00   apps  Deployment     default  banct-northrow-integration-svc-svc        Synced  Degraded              
2024-10-11T01:30:44+03:00   apps  Deployment     default  banct-northrow-integration-svc-svc        Synced  Degraded              deployment.apps/banct-northrow-integration-svc-svc unchanged
2024-10-11T01:30:44+03:00            Service     default  banct-northrow-integration-svc-service    Synced  Healthy               service/banct-northrow-integration-svc-service unchanged

Name:               argo-cd/banct-northrow-integration-svc
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                https://argocd.banct-app.com/applications/banct-northrow-integration-svc
Source:
- Repo:             https://gitlab.com/b5346/banct-devops.git
  Target:           main
  Path:             helm-deployment/api-helm
SyncWindow:         Sync Allowed
Sync Policy:        Automated
Sync Status:        Synced to main (abb1661)
Health Status:      Degraded

Operation:          Sync
Sync Revision:      abb16617a31f0ea9d012d5904b3f2923da9f7569
Phase:              Succeeded
Start:              2024-10-11 01:30:43 +0300 +03
Finished:           2024-10-11 01:30:44 +0300 +03
Duration:           1s
Message:            successfully synced (all tasks run)

GROUP  KIND        NAMESPACE  NAME                                    STATUS  HEALTH    HOOK  MESSAGE
       Service     default    banct-northrow-integration-svc-service  Synced  Healthy         service/banct-northrow-integration-svc-service unchanged
apps   Deployment  default    banct-northrow-integration-svc-svc      Synced  Degraded        deployment.apps/banct-northrow-integration-svc-svc unchanged


dell@dell-Latitude-3420:~/Downloads/argo-cd-7.6.8/argo-cd$ kubectl get secret -n argo-cd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
nxxxxxxxxxxxx

