# 🌳 ConfigMaps & Secrets – Mind Map

```text
                         CONFIGMAPS & SECRETS
                                   │
        ┌──────────────────────────┴──────────────────────────┐
        │                                                     │
   CONFIGMAP                                             SECRET
 (Non-Sensitive)                                      (Sensitive)
        │                                                     │
        │                                                     │
  Examples:                                              Examples:
  - App Config                                           - Passwords
  - URLs                                                 - API Keys
  - Feature Flags                                        - Tokens
  - Ports                                                - Certificates
        │                                                     │
        └──────────────────────────┬──────────────────────────┘
                                   │
                           CREATE METHODS
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         │                         │                         │
      Literal                    File                      YAML
         │                         │                         │
 kubectl create          --from-file=...          apiVersion: v1
 configmap/secret                               kind: ConfigMap/Secret

─────────────────────────────────────────────────────────────────────

                        CONSUME IN POD
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         │                         │                         │
       env                     envFrom                  Volume
         │                         │                         │
   Single Key                All Keys                Files Mounted
         │                         │                         │
FIRSTNAME=rahul            Multiple Env         /etc/config/key1
                                                   /etc/config/key2

─────────────────────────────────────────────────────────────────────

                        CONFIGMAP EXAMPLES
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         │                         │                         │
   configMapKeyRef          configMapRef              Volume Mount
         │                         │                         │
 env:                    envFrom:                 volumes:
 - name: APP             - configMapRef             configMap:
   valueFrom:              name: myapp                name: myapp

─────────────────────────────────────────────────────────────────────

                         SECRET EXAMPLES
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         │                         │                         │
   secretKeyRef             secretRef                 Volume Mount
         │                         │                         │
 env:                    envFrom:                 volumes:
 - name: PASS            - secretRef                secret:
   valueFrom:              name: db-secret            secretName: db-secret

─────────────────────────────────────────────────────────────────────

                       UPDATE BEHAVIOR
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         │                         │                         │
        env                    envFrom                  Volume
         │                         │                         │
      NO AUTO                  NO AUTO                 AUTO UPDATE
      UPDATE                   UPDATE                  (few seconds)
         │                         │                         │
  Restart Pod Needed      Restart Pod Needed      No Restart Needed

─────────────────────────────────────────────────────────────────────

                      VERIFICATION COMMANDS
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         │                         │                         │
      Env Vars                 Volume Files            Debug
         │                         │                         │
 printenv                 ls /etc/config          kubectl describe
 echo $VAR                cat file                kubectl get -o yaml

─────────────────────────────────────────────────────────────────────

                         EXAM TRAPS 🚨
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         │                         │                         │
 Wrong Case               Wrong Indent            Wrong Verify
         │                         │                         │
 configMapKeyRef       YAML nesting error      printenv won't show
 secretKeyRef                                volume-mounted values

─────────────────────────────────────────────────────────────────────

                     CONFIGMAP vs SECRET
                                   │
      ┌────────────────────────────┴───────────────────────────┐
      │                                                        │
  ConfigMap                                               Secret
      │                                                        │
 Plain Text                                         Base64 Encoded
 Non-Sensitive                                      Sensitive Data
 App Config                                         Credentials
 Easier Debugging                                   More Secure
```

## 🎯 30-Second Interview Revision

**ConfigMap**

* Stores non-sensitive configuration.
* Used via `env`, `envFrom`, or `volume`.
* Volume updates automatically.
* Env variables require pod restart after updates.

**Secret**

* Stores sensitive data like passwords, tokens, certificates.
* Same consumption methods as ConfigMap.
* Data stored as Base64 encoded values.
* Prefer mounting as volume for credentials.

**Golden Rule for CKA**

> ConfigMap = Configuration
> Secret = Credentials
> Env = Restart Required
> Volume = Auto Refresh

This mind map covers about **90% of ConfigMap & Secret questions** typically asked in CKA and 0–3 year DevOps/Kubernetes interviews. 🚀
