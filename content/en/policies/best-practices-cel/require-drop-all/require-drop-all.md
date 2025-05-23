---
title: "Drop All Capabilities in CEL expressions"
category: Best Practices in CEL
version: 1.11.0
subject: Pod
policyType: "validate"
description: >
    Capabilities permit privileged actions without giving full root access. All capabilities should be dropped from a Pod, with only those required added back. This policy ensures that all containers explicitly specify the `drop: ["ALL"]` ability. Note that this policy also illustrates how to cover drop entries in any case although this may not strictly conform to the Pod Security Standards.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//best-practices-cel/require-drop-all/require-drop-all.yaml" target="-blank">/best-practices-cel/require-drop-all/require-drop-all.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: drop-all-capabilities
  annotations:
    policies.kyverno.io/title: Drop All Capabilities in CEL expressions
    policies.kyverno.io/category: Best Practices in CEL 
    policies.kyverno.io/severity: medium
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/minversion: 1.11.0
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Capabilities permit privileged actions without giving full root access. All
      capabilities should be dropped from a Pod, with only those required added back.
      This policy ensures that all containers explicitly specify the `drop: ["ALL"]`
      ability. Note that this policy also illustrates how to cover drop entries in any
      case although this may not strictly conform to the Pod Security Standards.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: require-drop-all
      match:
        any:
        - resources:
            kinds:
              - Pod
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          variables:
            - name: allContainers
              expression: "object.spec.containers + object.spec.?initContainers.orValue([]) + object.spec.?ephemeralContainers.orValue([])"
          expressions:
            - expression: >-
                variables.allContainers.all(container, 
                container.?securityContext.?capabilities.?drop.orValue([]).exists(capability, capability.upperAscii() == 'ALL'))
              message: "Containers must drop `ALL` capabilities."


```
