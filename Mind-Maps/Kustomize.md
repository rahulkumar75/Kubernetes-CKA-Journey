# Kustomize (Kubernetes) – Mind Map
```text
                                ┌─────────────────────┐
                                │     KUSTOMIZE       │
                                └──────────┬──────────┘
                                           │
         ┌─────────────────────────────────┼─────────────────────────────────┐
         │                                 │                                 │
         ▼                                 ▼                                 ▼

 ┌─────────────────┐             ┌─────────────────┐              ┌─────────────────┐
 │   WHAT IS IT?   │             │   WHY USE IT?   │              │ CORE FILES      │
 └─────────────────┘             └─────────────────┘              └─────────────────┘
         │                                 │                                 │
         ├─ Native K8s config              ├─ Environment specific           ├─ kustomization.yaml
         │  customization                  │  deployments                   ├─ deployment.yaml
         ├─ No templates                   ├─ Avoid YAML duplication         ├─ service.yaml
         ├─ Built into kubectl             ├─ Easier maintenance             └─ other manifests
         └─ Declarative approach           └─ Cleaner than manual copies


                                           │
                                           ▼

                          ┌────────────────────────────┐
                          │    KUSTOMIZATION.YAML      │
                          └─────────────┬──────────────┘
                                        │
      ┌─────────────────────────────────┼──────────────────────────────────┐
      │                                 │                                  │
      ▼                                 ▼                                  ▼

┌───────────────┐             ┌─────────────────┐              ┌─────────────────┐
│ resources     │             │ commonLabels    │              │ namespace       │
└───────────────┘             └─────────────────┘              └─────────────────┘
      │                               │                                │
      ├─ deployment.yaml              ├─ Added to all objects          └─ Sets namespace
      ├─ service.yaml                 └─ Useful for grouping              automatically
      └─ configmap.yaml


      ▼
┌─────────────────────────────────────────────────────────┐
│ Example                                                  │
│ resources:                                               │
│   - deployment.yaml                                      │
│   - service.yaml                                         │
└─────────────────────────────────────────────────────────┘



                 ┌─────────────────────────────────┐
                 │      NAME MODIFICATIONS         │
                 └──────────────┬──────────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼

 ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
 │ namePrefix   │      │ nameSuffix   │      │ Result       │
 └──────────────┘      └──────────────┘      └──────────────┘
        │                     │                     │
        ├─ dev-              ├─ -v1               └─ dev-nginx-v1
        └─ prod-             └─ -prod


                                │
                                ▼

                 ┌─────────────────────────────────┐
                 │       CONFIGMAP & SECRET        │
                 └──────────────┬──────────────────┘
                                │
         ┌──────────────────────┼──────────────────────┐
         │                      │                      │
         ▼                      ▼                      ▼

 ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
 │ ConfigMapGen │      │ SecretGen    │      │ Auto Hash    │
 └──────────────┘      └──────────────┘      └──────────────┘
         │                     │                     │
         ├─ literals           ├─ literals          ├─ Rolling updates
         ├─ files              ├─ files             └─ Version tracking
         └─ env files          └─ env files


                                │
                                ▼

                 ┌─────────────────────────────────┐
                 │          PATCHES                │
                 └──────────────┬──────────────────┘
                                │
      ┌─────────────────────────┼─────────────────────────┐
      │                         │                         │
      ▼                         ▼                         ▼

┌────────────────┐    ┌────────────────┐     ┌────────────────┐
│ Strategic Merge│    │ JSON 6902      │     │ Use Cases      │
└────────────────┘    └────────────────┘     └────────────────┘
       │                       │                      │
       ├─ Partial YAML         ├─ Precise edits      ├─ Change image
       ├─ Easy to read         ├─ add/remove ops     ├─ Add replicas
       └─ Most common          └─ Advanced patching  └─ Update labels


                                │
                                ▼

                 ┌─────────────────────────────────┐
                 │      BASE & OVERLAYS            │
                 └──────────────┬──────────────────┘
                                │

          ┌─────────────────────┴─────────────────────┐
          │                                           │
          ▼                                           ▼

 ┌─────────────────┐                       ┌─────────────────┐
 │ Base            │                       │ Overlays        │
 └─────────────────┘                       └─────────────────┘
          │                                           │
          ├─ Common manifests                         ├─ Dev
          ├─ Shared configuration                     ├─ QA
          └─ Reusable                                 ├─ Staging
                                                      └─ Production


Example Structure

project/
│
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
│
└── overlays/
    ├── dev/
    ├── qa/
    └── prod/


                                │
                                ▼

                 ┌─────────────────────────────────┐
                 │        IMPORTANT COMMANDS       │
                 └──────────────┬──────────────────┘
                                │

     kubectl kustomize .

     kubectl apply -k .

     kubectl delete -k .

     kubectl diff -k .



                                │
                                ▼

                 ┌─────────────────────────────────┐
                 │ INTERVIEW QUESTIONS (0-3 YRS)   │
                 └──────────────┬──────────────────┘
                                │
      ├─ What is Kustomize?
      ├─ Difference between Helm and Kustomize?
      ├─ What are Bases and Overlays?
      ├─ How do patches work?
      ├─ How to generate ConfigMaps?
      ├─ What is namePrefix/nameSuffix?
      ├─ How do you deploy using Kustomize?
      └─ Why is Kustomize preferred for simple environments?


                                │
                                ▼

                 ┌─────────────────────────────────┐
                 │ HELM vs KUSTOMIZE               │
                 └─────────────────────────────────┘

      HELM                           KUSTOMIZE
      ────                           ─────────
      Uses Templates                 No Templates
      Package Manager                Config Customizer
      values.yaml                    overlays
      Complex Apps                   Simple/Medium Apps
      Helm Charts                    Native Kubernetes
      More Powerful                  Simpler
```

## 30-Second Interview Answer

**"Kustomize is a native Kubernetes configuration management tool that allows us to customize Kubernetes manifests without using templates. It works using a Base and Overlay model, where the base contains common resources and overlays contain environment-specific changes such as replica count, image versions, labels, namespaces, and patches. Kustomize is built into kubectl and is commonly used to manage dev, staging, and production deployments while avoiding YAML duplication."**

### Must Remember Keywords

* Base & Overlay
* Patches
* ConfigMapGenerator
* SecretGenerator
* namePrefix / nameSuffix
* Built into `kubectl`
* No Templates
* Environment-specific deployments
* `kubectl apply -k .`
* Alternative to Helm for simpler use cases

This is usually enough depth for **CKA + Junior DevOps Engineer (0–3 years)** interviews.
