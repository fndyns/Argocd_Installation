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


dell@dell-Latitude-3420:~/Downloads/argo-cd-7.6.8/argo-cd$ kubectl get secret -n argo-cd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
nxxxxxxxxxxxx

