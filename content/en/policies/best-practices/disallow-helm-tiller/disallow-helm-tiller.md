---
title: "Disallow Helm Tiller"
category: Sample
version: 1.6.0
subject: Pod
policyType: "validate"
description: >
    Tiller, found in Helm v2, has known security challenges. It requires administrative privileges and acts as a shared resource accessible to any authenticated user. Tiller can lead to privilege escalation as restricted users can impact other users. It is recommended to use Helm v3+ which does not contain Tiller for these reasons. This policy validates that there is not an image containing the name `tiller`.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//best-practices/disallow-helm-tiller/disallow-helm-tiller.yaml" target="-blank">/best-practices/disallow-helm-tiller/disallow-helm-tiller.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-helm-tiller
  annotations:
    policies.kyverno.io/title: Disallow Helm Tiller
    policies.kyverno.io/category: Sample
    policies.kyverno.io/minversion: 1.6.0
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Tiller, found in Helm v2, has known security challenges. It requires administrative privileges and acts as a shared
      resource accessible to any authenticated user. Tiller can lead to privilege escalation as
      restricted users can impact other users. It is recommended to use Helm v3+ which does not contain
      Tiller for these reasons. This policy validates that there is not an image
      containing the name `tiller`.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: validate-helm-tiller
      match:
        any:
        - resources:
            kinds:
              - Pod
      validate:
        message: "Helm Tiller is not allowed"
        foreach:
        - list: "request.object.spec.containers"
          pattern:
            image: "!*tiller*"
        - list: "request.object.spec.initContainers"
          pattern:
            image: "!*tiller*"
        - list: "request.object.spec.ephemeralContainers"
          pattern:
            image: "!*tiller*"

```
