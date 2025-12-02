SHELL := /usr/bin/env bash
DEV_NS ?= demo-1-hello-world
DEV_OVERLAY := apps/hello-aro/kustomize/overlays/dev
.PHONY: dev-apply dev-delete url
dev-apply: ; oc apply -k $(DEV_OVERLAY)
dev-delete: ; oc delete -k $(DEV_OVERLAY) --ignore-not-found
url: ; @oc get route hello -n $(DEV_NS) -o jsonpath='{.spec.host}{"\n"}'
