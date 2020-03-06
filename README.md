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
```

### Config

```yaml
prowjob_namespace: platform
pod_namespace: test-pods
log_level: debug

postsubmits:
  doulba/prow-release:
  - name: nightly-released
    branches:
    - ^release-$
    - ^v(\d+\.)?(\d+\.)?(\*|\d+)$
    spec:
      containers:
      - image: curlimages/curl
        command: ['curl','-X','POST','https://<JENKINS_URL>/job/prow/job/release/buildWithParameters?NUXEO_VERSION=$(PULL_BASE_REF)', '-u', '$(JENKINS_USER):$(JENKINS_PASSWORD)']
        env:
          - name: JENKINS_USER
            valueFrom:
              secretKeyRef:
                name: jenkins-token
                key: username
          - name: JENKINS_PASSWORD
            valueFrom:
              secretKeyRef:
                name: jenkins-token
                key: password
        securityContext:
          privileged: true
```
