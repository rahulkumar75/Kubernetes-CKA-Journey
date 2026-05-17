This NetworkPolicy allows only **frontend pods** to access **backend pods** on **TCP port 8080** inside the **ecommerce** namespace.

### Simple Interview Explanation:

* `podSelector: app=backend` → Policy applies to backend pods.
* `Ingress` → Controls incoming traffic to backend.
* `from: app=frontend` → Only frontend pods can connect.
* `port: 8080` → Access allowed only on port 8080.

### Final Meaning:

Frontend → Backend:8080 ✅ Allowed
Other Pods → Backend ❌ Blocked

### Why We Use It:

To secure internal communication and ensure only required services talk to each other (least privilege model).
