                         KUBERNETES CLUSTER
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
   CLUSTER INFO            CONTEXTS                NODES
        │                       │                       │
        │                       │                       │
kubectl cluster-info    kubectl config         kubectl get nodes
                        get-contexts
        │                       │                       │
        │                       │                       │
kubectl config         kubectl config         kubectl describe
get-clusters           current-context        node node-name
        │                       │
        │                       │
                        kubectl config
                        use-context ctx-name

────────────────────────────────────────────────────────────

                         API DISCOVERY
                                │
                ┌───────────────┴───────────────┐
                │                               │
                ▼                               ▼

      kubectl api-resources        kubectl api-versions

      (What resources exist?)      (Supported API versions)

────────────────────────────────────────────────────────────

                         DEBUGGING
                                │
                ┌───────────────┴───────────────┐
                │                               │
                ▼                               ▼

      kubectl get componentstatuses
      (legacy)

      kubectl get events
      kubectl get pods -A
      kubectl get all -A

────────────────────────────────────────────────────────────

                     EXAM QUICK FLOW
                                │
                                ▼

      1. Check active cluster
         kubectl config current-context

                    │
                    ▼

      2. Verify cluster health
         kubectl cluster-info

                    │
                    ▼

      3. Check nodes
         kubectl get nodes

                    │
                    ▼

      4. Investigate node
         kubectl describe node <node>

                    │
                    ▼

      5. Explore APIs
         kubectl api-resources