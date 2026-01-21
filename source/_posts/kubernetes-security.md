---
title: kubernetes security
date: 2026-01-22 00:16:26
tags: K8s
category: security
---


### Pod Security Admission (PSA)
Enforce a pod to follow a certain security context, otherwise it cannot be scheduled:

``` yaml

apiVersion: v1
kind: Namespace
metadata:
  labels:
    kubernetes.io/metadata.name: security_enforced
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: latest
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/warn-version: latest
  name: security_enforced
```

If you have pod violate the security policy, when restarting it will have following error:

``` bash

     kubectl rollout restart -f nginx.yaml 
Warning: would violate PodSecurity "restricted:latest": allowPrivilegeEscalation != false (container "nginx" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "nginx" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "nginx" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "nginx" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
deployment.apps/nginx restarted
```