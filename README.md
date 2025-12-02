# OpenShift-with-SAP — EIC North/South Demo

Repeatable demo of **SAP Integration Suite – Edge Integration Cell (EIC)** on **Red Hat OpenShift (ARO)** with a **north side** (SAP/BTP) and a **south side** (edge/disconnected systems).  
**First demo:** connect **Lenel** gear in the Houston lab (far south edge) and prove **north→south** and **south→north** message flow.

---

## Objectives
- Stand up **EIC runtime** on OpenShift (managed from **SAP BTP**).
- Provide a **pluggable southbound connector** pattern (Lenel first, others later).
- Demonstrate **bi-directional messaging** with simple validation steps.
- Keep it **repeatable** and **GitOps-friendly** (Kustomize overlays, optional Argo CD).

---

## Topology (text diagram)

```
NORTH (SAP BTP – Control Plane)
┌──────────────────────────────────────────────────────────────────────────┐
│  Global Account/Subaccount                                               │
│  └─ Integration Suite (manages EIC)                                      │
└──────────────────────────────────────────────────────────────────────────┘

Red Hat OpenShift (ARO) – EIC Runtime & South Connectors
┌──────────────────────────────────────────────────────────────────────────┐
│  Edge Integration Cell (EIC)                                             │
│  ┌───────────────┐  ┌───────────────┐                                   │
│  │ Inbound API   │  │ Outbound API  │                                   │
│  └───────────────┘  └───────────────┘                                   │
│     │                       │                                            │
│     │ Routes/TLS/Auth/Probes/Resources (platform bits)                   │
│     └────────────────────────────────────────────────────────────────────│
│     Mounts: ConfigMap(HTML) • Secrets(registry) • RBAC/SA                │
└──────────────────────────────────────────────────────────────────────────┘

South Connectors
┌──────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────┐   ┌───────────────────────────────┐     │
│  │ Lenel Connector (Houston)  │   │ MQTT/Kafka Bridges            │     │
│  │ HTTP/Webhook default       │   │ (AMQ Streams / Mosquitto)     │     │
│  └─────────────────────────────┘   └───────────────────────────────┘     │
│  Extensible adapters • per-site env/secrets • health endpoints           │
└──────────────────────────────────────────────────────────────────────────┘

FAR SOUTH EDGE — Houston Lab
┌───────────────────────────────┐
│ Lenel Hardware / APIs         │
└───────────────────────────────┘

Legend:  → request/command    ⇢ event/response (asynchronous)
```

---

## North↔South Message Sequence (text)

```
Actors:  [BTP: Integration Suite]   [OpenShift: EIC]   [Lenel Connector]

North → South
1) iFlow call        ───────────────→  EIC Inbound
2) Forward to Lenel  ───────────────────────────────→  Lenel Connector
3) Ack/Result        ⇠═══════════════════════════════  (async/response)
4) Response to iFlow ⇐═══════════════  EIC → BTP

South → North
5) Lenel Event       ───────────────────────────────→  EIC Endpoint
6) EIC → iFlow       ───────────────→  BTP Endpoint
7) Final 200 OK      ⇐═══════════════  (processed)

Notes:
- Solid arrows = synchronous HTTP(s) request/command
- Double-line arrows (═) = asynchronous event/response
```

---

## Repo layout (initial)
```
apps/
  eic-bootstrap/                 # EIC installer glue (values template, secrets templates)
  south-connectors/
    lenel/                       # Lenel connector (containerized)
  hello-aro/                     # tiny httpd “hello” for smoke tests
docs/
  images/                        # screenshots for README/runbook (add later)
```

## Prereqs (BTP side)
- BTP global account + subaccount
- Entitlements: **Integration Suite** + **Edge Integration Cell**
- Integration Suite **subscribed**; **EIC activated**
- Container registry (e.g., ACR) configured during EIC setup
- **Download EIC installer artifacts** (values/secrets) for OpenShift

## Quick start (cluster once artifacts are ready)
```bash
# login + namespace
oc whoami && oc version
oc new-project sap-eic || oc project sap-eic

# registry pull secret (example for ACR)
oc -n sap-eic create secret docker-registry eic-regcred \
  --docker-server=<acr>.azurecr.io \
  --docker-username=<acr-name> \
  --docker-password=<acr-password-or-token>

# install via Helm using your EIC values from BTP
helm upgrade --install eic ./apps/eic-bootstrap/chart \
  -n sap-eic \
  -f ./apps/eic-bootstrap/values.yaml \
  --set imageCredentials.secretName=eic-regcred \
  --set global.storageClass=azurefile-csi
```

## Validate
```bash
oc -n sap-eic get pods
oc -n sap-eic get svc,route
```

## Next steps
- Add `apps/eic-bootstrap/values-template.yaml` and secret templates
- Add `apps/south-connectors/lenel` skeleton + health endpoints
- Capture real screenshots under `docs/images/` and link into the README/runbook
- Optional: MQTT/Kafka transport and Argo CD GitOps
