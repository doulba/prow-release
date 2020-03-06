# prow-release

## Install Prow

> Pre-requisite install [bazel](https://docs.bazel.build/versions/master/install.html)

### create new namespace

```bash
kubectl create ns platform
kubectl config set-context --current --namespace=platform
```

### create clusterrolebinding

```bash
kubectl create clusterrolebinding cluster-admin-binding   --clusterrole cluster-admin --user $(gcloud config get-value account)
```

### create secret

```bash
kubectl create secret generic hmac-token --from-file=hmac=<FULL_PATH>/hmac-token
kubectl create secret generic oauth-token --from-file=oauth=<FULL_PATH>/github-oauth-token
```

### Deploy prow

```bash
kubectl apply -f <FULL_PATH>/prow/cluster/starter.yaml
```


### Plugins

```yaml
---
triggers:
- repos:
  - doulba/prow-release
  only_org_members: true


plugins:
  doulba/prow-release:
  - size
  - trigger

external_plugins:
  doulba/prow-release:
  - name: nightly-released
    endpoint: https://jekins-url/job/prow/job/release/build?token=<git-token>
    events:
    - create
```
