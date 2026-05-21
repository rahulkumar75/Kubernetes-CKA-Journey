# Real-Time CRD + CR + Controller Lab

Step-by-Step Guide

Step 1: Clone the sample-controller Repository
First, clone the kubernetes/sample-controller repository to your local machine and navigate into its directory.
```
git clone https://github.com/kubernetes/sample-controller
cd sample-controller
```
Step 2: Build and Run the Sample Controller
Build the controller executable and then run it. The controller will watch for Foo custom resources and create Deployments based on them.

### Ensure you have go client installed
```
sudo apt-get update -y
sudo apt install golang-go -y

sed -i 's/go 1\.24\.0/go 1.18/' go.mod
sed -i '/^godebug/d' go.mod
go mod tidy
```

Build the sample-controller from the source code
```
go build -o sample-controller .
```

### Run the sample-controller.

Keep this running in a separate terminal or in the background.
```
./sample-controller -kubeconfig=$HOME/.kube/config
```

> Note: Keep this process running in a separate terminal window or run it in the background, as it needs to continuously monitor for changes to Foo resources.



## Goal

Implement a complete real-world flow:

```text id="e3sn6u"
CRD        → API Extension
CR         → Desired State
Controller → Reconciliation
Status     → Actual State
```

We will simulate a real production-style use case:

# Scenario

Create a custom Kubernetes resource called:

```text id="jlwm2r"
WebApp
```

When user creates:

```yaml id="jlwm8p"
kind: WebApp
```

our controller logic (simulated manually) will:

* create a Deployment
* create a Service
* update status

Exactly how real operators work.

---

# Architecture

```text id="n3vjlwm"
            User
              |
              v
         Create WebApp CR
              |
              v
        Kubernetes API Server
              |
              v
      WebApp stored in etcd
              |
              v
      Controller watches CR
              |
      ---------------------
      |                   |
      v                   v
 Create Deployment    Create Service
              |
              v
      Update CR Status
```

---

# Lab Structure

| Step | Purpose                            |
| ---- | ---------------------------------- |
| 1    | Create CRD                         |
| 2    | Verify API extension               |
| 3    | Create CR                          |
| 4    | Simulate controller reconciliation |
| 5    | Create Deployment                  |
| 6    | Create Service                     |
| 7    | Update status                      |
| 8    | Observe desired vs actual state    |

---

# Prerequisites

* Kubernetes cluster
* kubectl
* Kind / Minikube recommended

---

# STEP 1 — Create CRD

## `webapp-crd.yaml`

```yaml id="jlwm1x"
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition

metadata:
  name: webapps.mycompany.com

spec:
  group: mycompany.com

  names:
    plural: webapps
    singular: webapp
    kind: WebApp
    shortNames:
      - wa

  scope: Namespaced

  versions:
    - name: v1
      served: true
      storage: true

      schema:
        openAPIV3Schema:
          type: object

          properties:
            spec:
              type: object

              required:
                - image
                - replicas

              properties:
                image:
                  type: string

                replicas:
                  type: integer
                  minimum: 1

                port:
                  type: integer
                  default: 80

            status:
              type: object

              properties:
                phase:
                  type: string

                availableReplicas:
                  type: integer

      subresources:
        status: {}

      additionalPrinterColumns:
        - name: Image
          type: string
          jsonPath: .spec.image

        - name: Replicas
          type: integer
          jsonPath: .spec.replicas

        - name: Phase
          type: string
          jsonPath: .status.phase
```

---

# Apply CRD

```bash id="jlwm3m"
kubectl apply -f webapp-crd.yaml
```

---

# Verify CRD

```bash id="jlwm7t"
kubectl get crd
```

---

# Verify API Extension

```bash id="jlwm4o"
kubectl api-resources | grep webapp
```

Expected:

```text id="jlwm8n"
webapps   wa   mycompany.com/v1
```

---

# IMPORTANT UNDERSTANDING

At this point:

```text id="jlwm9u"
CRD → API Extension Complete
```

Kubernetes now understands:

* WebApp resource
* new API endpoint
* kubectl operations

---

# STEP 2 — Create Custom Resource (Desired State)

## `my-webapp.yaml`

```yaml id="jlwm0k"
apiVersion: mycompany.com/v1
kind: WebApp

metadata:
  name: nginx-app

spec:
  image: nginx
  replicas: 2
  port: 80
```

---

# Apply CR

```bash id="jlwm6v"
kubectl apply -f my-webapp.yaml
```

---

# Verify

```bash id="jlwm1z"
kubectl get webapps
```

Expected:

```text id="jlwm5y"
NAME        IMAGE   REPLICAS   PHASE
nginx-app   nginx   2
```

---

# IMPORTANT UNDERSTANDING

This CR represents:

```text id="jlwm7a"
Desired State
```

User is saying:

```text id="jlwm9j"
"I want 2 nginx replicas running."
```

BUT:

* no pods exist yet
* no deployment exists yet

Because:

* controller/operator is missing

---

# STEP 3 — Simulate Controller Reconciliation

In real world:

* Operator watches CR
* Automatically creates resources

We will manually simulate it.

---

# Check Current State

```bash id="jlwm2s"
kubectl get deploy
kubectl get svc
```

Expected:

```text id="jlwm4d"
No resources found
```

---

# STEP 4 — Controller Creates Deployment

## `deployment.yaml`

```yaml id="jlwm5q"
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: nginx-app

  template:
    metadata:
      labels:
        app: nginx-app

    spec:
      containers:
        - name: nginx
          image: nginx

          ports:
            - containerPort: 80
```

---

# Apply Deployment

```bash id="jlwm0n"
kubectl apply -f deployment.yaml
```

---

# Verify Pods

```bash id="jlwm6e"
kubectl get pods
```

---

# IMPORTANT UNDERSTANDING

This step represents:

```text id="jlwm7i"
Controller → Reconciliation
```

Controller saw:

| Desired    | Actual     |
| ---------- | ---------- |
| 2 replicas | 0 replicas |

So it created Deployment.

---

# STEP 5 — Controller Creates Service

## `service.yaml`

```yaml id="jlwm8l"
apiVersion: v1
kind: Service

metadata:
  name: nginx-app

spec:
  selector:
    app: nginx-app

  ports:
    - port: 80
      targetPort: 80

  type: ClusterIP
```

---

# Apply

```bash id="jlwm4h"
kubectl apply -f service.yaml
```

---

# Verify

```bash id="jlwm3b"
kubectl get svc
```

---

# IMPORTANT UNDERSTANDING

Controller now ensured:

* application reachable
* networking configured

---

# STEP 6 — Update CR Status (Actual State)

Real operators update:

```yaml id="jlwm7o"
status:
```

to show actual cluster state.

---

# Patch Status

```bash id="jlwm1q"
kubectl patch webapp nginx-app \
  --subresource=status \
  --type=merge \
  -p '{"status":{"phase":"Running","availableReplicas":2}}'
```

---

# Verify

```bash id="jlwm5c"
kubectl get webapps
```

Expected:

```text id="jlwm9v"
NAME        IMAGE   REPLICAS   PHASE
nginx-app   nginx   2          Running
```

---

# Full YAML Output

```bash id="jlwm8w"
kubectl get webapp nginx-app -o yaml
```

Observe:

```yaml id="jlwm0a"
spec:
  image: nginx
  replicas: 2

status:
  phase: Running
  availableReplicas: 2
```

---

# FINAL UNDERSTANDING

---

# 1. CRD → API Extension

```text id="jlwm6n"
Kubernetes learned a new resource type:
WebApp
```

---

# 2. CR → Desired State

```yaml id="jlwm2j"
spec:
  replicas: 2
```

User expectation.

---

# 3. Controller → Reconciliation

Controller compared:

| Desired    | Actual     |
| ---------- | ---------- |
| 2 replicas | 0 replicas |

Then fixed mismatch.

---

# 4. Status → Actual State

```yaml id="jlwm3x"
status:
  phase: Running
```

Represents:

* current live state

---

# Real Operator Flow

This is EXACTLY how:

* Argo CD
* Cert-Manager
* Prometheus Operator
* Database Operators

work internally.

---

# Bonus Practice Tasks

---

# Task 1 — Scale Application

Update CR:

```yaml id="jlwm4y"
replicas: 5
```

Then manually update Deployment.

Observe:

* desired vs actual

---

# Task 2 — Invalid Validation

Try:

```yaml id="jlwm7s"
replicas: 0
```

Expected:

* validation failure

---

# Task 3 — Add New Status

Patch:

```yaml id="jlwm9e"
status:
  url: nginx-app.default.svc.cluster.local
```

---

# Task 4 — Watch Events

Terminal 1:

```bash id="jlwm1m"
kubectl get webapps -w
```

Terminal 2:

* modify CR
* patch status

Observe changes live.

---

# Production-Level Mental Model

```text id="jlwm0z"
CRD        = API Extension
CR         = Desired State
Controller = Automation Brain
Status     = Actual State
```

This is one of the MOST important Kubernetes concepts for:

* Operators
* Platform Engineering
* GitOps
* Service Mesh
* Cloud Native systems
* Kubernetes internals

---

# Cleanup

```bash id="jlwm5n"
kubectl delete crd webapps.mycompany.com

kubectl delete deploy nginx-app

kubectl delete svc nginx-app
```

---

# Interview-Level Summary

> In this lab, the CRD extended the Kubernetes API with a new WebApp resource, the CR defined the desired application state, the controller reconciliation logic created Deployment and Service resources, and the status field reflected the actual running state inside the cluster.
