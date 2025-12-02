# Hello ARO (Kustomize)
Deploy an OpenShift-friendly hello app with UBI9 httpd (8080), Service (80->8080), Route (edge TLS).
- Base includes probes, resource requests/limits, non-root securityContext
- ConfigMap is mounted at /var/www/html; changes trigger rollout via configMapGenerator

## Dev deploy
oc apply -k apps/hello-aro/kustomize/overlays/dev
oc get route hello -n demo-1-hello-world -o jsonpath='{.spec.host}{"\n"}'

## GitOps
oc apply -f apps/hello-aro/gitops/application.yaml
