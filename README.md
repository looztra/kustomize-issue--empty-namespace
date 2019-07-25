# Kustomize v3.0.3 generates empty namespaces in some cases

## Requirements

- [kubesplit](https://github.com/looztra/kubesplit) installed (`pip3 install -U --user kubesplit` for instance)

## OK with kustomize whatever the version

`kustomize build component-sets/overlay-ok | kubesplit -p -q -c -o generated/ok`

Resulting descriptors can be seen in directory `generated/ok`

The clusterrolebinding and the rolebinding have the namespace correctly specified

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    company.com/kustomize-component: kube-state-metrics
    company.com/kustomize-component-set: monitoring
  name: company:kube-state-metrics
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: company:kube-state-metrics
subjects:
  - kind: ServiceAccount
    name: kube-state-metrics
    namespace: monitoring
```

## NOK with kustomize v3.0.3, OK with v3.0.2 and v2.1.0

`kustomize build component-sets/overlay-not-ok | kubesplit -p -q -c -o generated/not-ok`

Resulting descriptors can be seen in directory `generated/not-ok`

The clusterrolebinding and the rolebinding have an empty namespace

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    company.com/kustomize-component: kube-state-metrics
    company.com/kustomize-component-set: monitoring
  name: company:kube-state-metrics
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: company:kube-state-metrics
subjects:
  - kind: ServiceAccount
    name: kube-state-metrics
    namespace:
```

## What is the difference between the 2 use cases?

- OK => a configMapGenerator that generates a ConfigMap in another namespace as the one of the target ServiceAccount
- NOK => a configMapGenerator that generates a ConfigMap in the same namespace as the one of the target ServiceAccount
