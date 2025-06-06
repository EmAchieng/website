---
title: "Check Image Base"
category: Other, EKS Best Practices
version: 1.7.0
subject: Pod
policyType: "validate"
description: >
    Container images can be built from a variety of sources, including other preexisting images. Ensuring images that are allowed to run are built from known, trusted images where their provenance is guaranteed can be an important step in ensuring overall cluster security. This policy ensures that any container image specifies some base image in its metadata from four possible sources: Docker BuildKit, OCI annotations (in manifest or config), or Buildpacks. Note that the ability to detect the presence of a base image is not implicit and requires the author to specify it using metadata or build directives of some sort (ex., Dockerfile FROM statements do not automatically expose this information).
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/require-base-image/require-base-image.yaml" target="-blank">/other/require-base-image/require-base-image.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-base-image
  annotations:
    policies.kyverno.io/title: Check Image Base
    policies.kyverno.io/category: Other, EKS Best Practices
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.7.0
    policies.kyverno.io/minversion: 1.7.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Container images can be built from a variety of sources, including other preexisting
      images. Ensuring images that are allowed to run are built from known, trusted
      images where their provenance is guaranteed can be an important step in ensuring
      overall cluster security. This policy ensures that any
      container image specifies some base image in its metadata from four possible sources:
      Docker BuildKit, OCI annotations (in manifest or config), or Buildpacks. Note that the
      ability to detect the presence of a base image is not implicit and requires the author
      to specify it using metadata or build directives of some sort (ex., Dockerfile FROM
      statements do not automatically expose this information).
spec:
  validationFailureAction: Audit
  rules:
  - name: require-base-image
    match:
      any:
      - resources:
          kinds:
          - Pod
    preconditions:
      all:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: NotEquals
        value: DELETE
    validate:
      message: "Images must specify a source/base image from which they are built."
      foreach:
      - list: "request.object.spec.containers"
        context: 
        - name: imageData
          imageRegistry: 
            reference: "{{ element.image }}"
        - name: mobysource
          variable:
            jmesPath: imageData.configData."moby.buildkit.buildinfo.v1" | base64_decode(@).parse_json(@) | sources[?type == 'docker-image'].ref | length(@)
            default: 0
        - name: ocisourcelabels
          variable:
            jmesPath: imageData.configData.config.Labels | keys(@)
            default: []
        - name: ocisourceannotations
          variable:
            jmesPath: imageData.manifest.annotations."org.opencontainers.image.base.name"
            default: ''
        - name: buildpacks
          variable:
            jmesPath: parse_json(imageData.configData.config.Labels."io.buildpacks.lifecycle.metadata").runImage.reference || parse_json(imageData.configData.config.Labels."io.buildpacks.lifecycle.metadata").stack.runImage.image
            default: ''
        deny:
          conditions:
            all:
              - key: "org.opencontainers.image.base.name"
                operator: AnyNotIn
                value: "{{ ocisourcelabels}}"
              - key: "{{ ocisourceannotations}}"
                operator: Equals
                value: ''
              - key: "{{ mobysource }}"
                operator: Equals
                value: 0
              - key: "{{ buildpacks }}"
                operator: Equals
                value: ''

```
