---
title: "Disallow all Secrets"
category: Other
version: 1.6.0
subject: Pod, Secret
policyType: "validate"
description: >
    Secrets often contain sensitive information which not all Pods need consume. This policy disables the use of all Secrets in a Pod definition. In order to work effectively, this Policy needs a separate Policy or rule to require `automountServiceAccountToken=false` at the Pod level or ServiceAccount level since this would otherwise result in a Secret being mounted.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/disallow-all-secrets/disallow-all-secrets.yaml" target="-blank">/other/disallow-all-secrets/disallow-all-secrets.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: no-secrets
  annotations:
    policies.kyverno.io/title: Disallow all Secrets
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod, Secret
    kyverno.io/kyverno-version: 1.6.0
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.21"
    policies.kyverno.io/description: >-
      Secrets often contain sensitive information which not all Pods need consume.
      This policy disables the use of all Secrets in a Pod definition. In order to work effectively,
      this Policy needs a separate Policy or rule to require `automountServiceAccountToken=false`
      at the Pod level or ServiceAccount level since this would otherwise result in a Secret being mounted.
spec:
  validationFailureAction: Audit
  rules:
  - name: secrets-not-from-env
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "No Secrets from env."
      pattern:
        spec:
          =(ephemeralContainers):
          - name: "*"
            =(env):
            - =(valueFrom):
                X(secretKeyRef): "null"
          =(initContainers):
          - name: "*"
            =(env):
            - =(valueFrom):
                X(secretKeyRef): "null"
          containers:
          - name: "*"
            =(env):
            - =(valueFrom):
                X(secretKeyRef): "null"
  - name: secrets-not-from-envfrom
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "No Secrets from envFrom."
      pattern:
        spec:
          =(ephemeralContainers):
          - name: "*"
            =(envFrom):
            - X(secretRef): "null"
          =(initContainers):
          - name: "*"
            =(envFrom):
            - X(secretRef): "null"
          containers:
          - name: "*"
            =(envFrom):
            - X(secretRef): "null"
  - name: secrets-not-from-volumes
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "No Secrets from volumes."
      pattern:
        spec:
          =(volumes):
          - X(secret): "null"

```
