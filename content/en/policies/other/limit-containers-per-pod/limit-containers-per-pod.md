---
title: "Limit Containers per Pod"
category: Sample
version: 1.6.0
subject: Pod
policyType: "validate"
description: >
    Pods can have many different containers which are tightly coupled. It may be desirable to limit the amount of containers that can be in a single Pod to control best practice application or so policy can be applied consistently. This policy checks all Pods to ensure they have no more than four containers.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/limit-containers-per-pod/limit-containers-per-pod.yaml" target="-blank">/other/limit-containers-per-pod/limit-containers-per-pod.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: limit-containers-per-pod
  annotations:
    policies.kyverno.io/title: Limit Containers per Pod
    policies.kyverno.io/category: Sample
    policies.kyverno.io/minversion: 1.6.0
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Pods can have many different containers which
      are tightly coupled. It may be desirable to limit the amount of containers that
      can be in a single Pod to control best practice application or so policy can
      be applied consistently. This policy checks all Pods to ensure they have
      no more than four containers.
spec:
  validationFailureAction: Audit
  background: false
  rules:
  - name: limit-containers-per-pod
    match:
      any:
      - resources:
          kinds:
          - Pod
    preconditions:
      all:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: AnyIn
        value: 
        - CREATE
        - UPDATE
    validate:
      message: "Pods can only have a maximum of 4 containers."
      deny:
        conditions:
          any:
          - key: "{{request.object.spec.containers[] | length(@)}}"
            operator: GreaterThan
            value: "4"
```
